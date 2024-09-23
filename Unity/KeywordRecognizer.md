# Unityで簡単音声認識
めっちゃ簡単に音声認識ができた！
参照記事：https://learn.microsoft.com/ja-jp/windows/mixed-reality/develop/unity/voice-input-in-unity

## やりかた
1. コードを書いてどっかにくっつける（インスタンス化）
2. ```inspector``` からキーワードを ```words``` に登録
3. ```flags``` を参照して，言われたかどうかを確認する
4. 参照し終わったら ```flags``` は必ず ```false``` に戻す（じゃないとずっと反応する）

## 注意事項
- WebGLだと動かない！
- マイクの設定はする必要はない（そもそも記事に書いてる設定項目がない）
- クラスを継承するとなんかめっちゃ重くなる
- 音声認識がうまくいかないときは再起動すること！
- はっきりと話すと割と読み取ってくれる
- もっと精度を上げるにはお金はかかるがAzureの音声認識を使うといいみたい

## コード
```C#
using System.Collections;
using UnityEngine;
using UnityEngine.Windows.Speech;
using System.Collections.Generic;
using System.Linq;

public class SpeechController : MonoBehaviour
{
    public List<string> words = new List<string>();
    public Dictionary<string, bool> flags = new Dictionary<string, bool>();
    private KeywordRecognizer keywordRecognizer;
    private Dictionary<string, System.Action> keywords = new Dictionary<string, System.Action>();
    // Start is called before the first frame update
    void Start()
    {

        foreach (string word in words)
        {
            flags.Add(word, false);
            keywords.Add(word, () =>
            {
                Debug.Log("Keyword: " + word);
                try
                {
                    flags[word] = true;
                }
                catch (System.Exception e)
                {
                    Debug.Log(e);
                }
            });
        };

        keywordRecognizer = new KeywordRecognizer(keywords.Keys.ToArray());
        keywordRecognizer.OnPhraseRecognized += KeywordRecognizer_OnPhraseRecognized;
        keywordRecognizer.Start();
    }

    // Update is called once per frame
    void Update()
    {

    }

    private void KeywordRecognizer_OnPhraseRecognized(PhraseRecognizedEventArgs args)
    {
        System.Action keywordAction;
        if (keywords.TryGetValue(args.text, out keywordAction))
        {
            keywordAction.Invoke();
        }
    }
}
```
