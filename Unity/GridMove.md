# Unityでグリッド移動させる方法

下記記事を参考にすると実装できた！マジでありがたい．

【Unity】3Dローグライクゲームの作り方〜Step3-3〜｜みにに：https://note.com/frabbit_mbp/n/n90ceada318db

## Pos2Dの定義
めんどくさかったら別に作らなくてもよい
```C#
/**
* 2次元の座標クラス
*/
[System.Serializable]
public class Pos2D
{
   public int x, z;
}
```
## 方向指定
これは便利！
```C#
/**
* 方向を表す定数
* 今回は4方向のみをサポートします
*/
public enum EDir
{
   Pause, // 静止(NULLの代わり)
   Left,  // 左
   Up,    // 上
   Right, // 右
   Down,  // 下
};
```
## プレイヤーの移動
ここがメイン．それぞれの解説は元記事を参照されたし．

フレームごとに少しずつ移動させるのが肝．
```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerGridMove : MonoBehaviour
{
    public Pos2D grid;
    private Pos2D newGrid = null;
    public EDir direction;
    public float speed = 0.9f;
    public float maxPerFrame = 1.67f;
    public float completeFrame;
    private int currentFrame = 0;
    // Start is called before the first frame update
    void Start()
    {
        completeFrame = maxPerFrame / Time.deltaTime;
    }

    // Update is called once per frame
    void Update()
    {
        if (currentFrame == 0)
        {
            EDir d = KeyToDir();
            if (d != EDir.Pause)
            {
                direction = d;
                transform.rotation = DirToRotation(direction);
                newGrid = GetNewGrid(grid, direction);
                grid = Move(grid, newGrid, ref currentFrame);
            }
        }
        else
        {
            grid = Move(grid, newGrid, ref currentFrame);
        }
    }

    /**
   * 入力されたキーに対応する向きを返す
   */
    public virtual EDir KeyToDir()
    {
        if (!Input.anyKey)
        {
            Debug.Log("Pause");
            return EDir.Pause;
        }
        if (Input.GetKeyDown(KeyCode.LeftArrow))
        {
            Debug.Log("Left");
            return EDir.Left;
        }
        if (Input.GetKeyDown(KeyCode.UpArrow))
        {
            Debug.Log("Up");
            return EDir.Up;
        }
        if (Input.GetKeyDown(KeyCode.RightArrow))
        {
            Debug.Log("Right");
            return EDir.Right;
        }
        if (Input.GetKeyDown(KeyCode.DownArrow))
        {
            Debug.Log("Down");
            return EDir.Down;
        }
        return EDir.Pause;
    }

    /**
   * 引数で与えられた向きに対応する回転のベクトルを返す
   */
    public virtual Quaternion DirToRotation(EDir d)
    {
        Quaternion r = Quaternion.Euler(0, 0, 0);
        switch (d)
        {
            case EDir.Left:
                r = Quaternion.Euler(0, 270, 0); break;
            case EDir.Up:
                r = Quaternion.Euler(0, 0, 0); break;
            case EDir.Right:
                r = Quaternion.Euler(0, 90, 0); break;
            case EDir.Down:
                r = Quaternion.Euler(0, 180, 0); break;
        }
        return r;
    }

    /**
* グリッド座標をワールド座標に変換
*/
    private float ToWorldX(int xgrid)
    {
        return xgrid * 2;
    }

    private float ToWorldZ(int zgrid)
    {
        return zgrid * 2;
    }

    /**
    * ワールド座標をグリッド座標に変換
*/
    private int ToGridX(float xworld)
    {
        return Mathf.FloorToInt(xworld / 2);
    }

    private int ToGridZ(float zworld)
    {
        return Mathf.FloorToInt(zworld / 2);
    }

    /**
    * 補完で計算して進む
*/
    private Pos2D Move(Pos2D currentPos, Pos2D newPos, ref int frame)
    {
        float px1 = ToWorldX(currentPos.x);
        float pz1 = ToWorldZ(currentPos.z);
        float px2 = ToWorldX(newPos.x);
        float pz2 = ToWorldZ(newPos.z);
        frame += 1;
        float t = frame / completeFrame;
        float newX = px1 + (px2 - px1) * t;
        float newZ = pz1 + (pz2 - pz1) * t;
        transform.position = new Vector3(newX, transform.position.y, newZ);
        if (frame >= completeFrame)
        {
            currentPos = newPos;
            frame = 0;
        }
        return currentPos;
    }

    /**
    * 現在の座標(position)と移動したい方向(d)を渡すと
    * 移動先の座標を取得
*/
    public virtual Pos2D GetNewGrid(Pos2D position, EDir direction)
    {
        Pos2D newPos = new Pos2D();
        newPos.x = position.x;
        newPos.z = position.z;
        switch (direction)
        {
            case EDir.Left:
                newPos.x -= 1; break;
            case EDir.Up:
                newPos.z += 1; break;
            case EDir.Right:
                newPos.x += 1; break;
            case EDir.Down:
                newPos.z -= 1; break;
        }
        return newPos;
    }

}
```
