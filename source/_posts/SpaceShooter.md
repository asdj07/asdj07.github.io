---
layout: post
title: SpaceShooter
date: 2016-09-09 15:31:22
tags: unity
---

根据Unity官网的SpaceShooter教程视频和参考资料的博客完成了这个小游戏，下边简单的记录一下。
参考资料：
http://www.jianshu.com/p/8cc3a2109d3b
https://unity3d.com/cn/learn/tutorials/projects/space-shooter-tutorial
***

<!-- more -->

![](http://upload-images.jianshu.io/upload_images/1730128-ce666a11e9ac35cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 开发步骤

* **1.**首先在Unity的 Asset Store下载游戏的资源包，包含了游戏的背景，模型，音频等。

* **2.**将飞机模型Player添加到新建的场景_Scenes中，调整相机位置，新建灯光，调整灯光位置。

* **3.**.为Player添加脚本PlayerController，控制Player移动。

```csharp

 void FixedUpdate()
    {
        float moveHorizontal = Input.GetAxis("Horizontal");
        float moveVertical  = Input.GetAxis("Vertical");

        //moveHorizontal和moveVertical记录玩家输入的方向数据
        Vector3 movement = new Vector3(moveHorizontal, 0.0f, moveVertical);
        //刚体根据矢量方向movement移动，位移为speed
        GetComponent<Rigidbody>().velocity = movement * speed;

        //移动时倾斜，以Z为中心轴旋转
        GetComponent<Rigidbody>().rotation = Quaternion.Euler(0.0f, 0.0f, GetComponent<Rigidbody>().velocity.x * - tilt);

        //首先序列化Boundary类,保存飞船在X和Z轴上的范围
        //Mathf.Clamp将飞船的x和z值锁定在屏幕范围内
        GetComponent<Rigidbody>().position = new Vector3(
            Mathf.Clamp(GetComponent<Rigidbody>().position.x, boundary.xMin, boundary.xMax),
            0.0f,
            Mathf.Clamp(GetComponent<Rigidbody>().position.z, boundary.zMin, boundary.zMax)
            );
    }
```

![](http://upload-images.jianshu.io/upload_images/1730128-51aded257343632c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **4.**添加游戏背景，制作子弹，为子弹添加碰撞：添加Rigidbody（diselect use gravity）， 添加碰撞Mesh Collider（select is Trigger），为子弹编写脚本Mover，使子弹发射，将子弹设为Prefab

```csharp
   public float speed;
	void Start () {
       //transform.forward:The blue axis of the transform in world space.
        GetComponent<Rigidbody>().velocity = transform.forward * speed;
    }

```

![](http://upload-images.jianshu.io/upload_images/1730128-fed3e60007508327.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **5.**设置火箭发射子弹。设置子弹发射点，新建**Shot Spawn**，把**Shot Spawn**拖入到Player下面，并调整**Shot Spawn**的位置到合适的点，在PlayerController编写脚本

![](http://upload-images.jianshu.io/upload_images/1730128-fb8c37ecfc5f6935.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```csharp
    private float nextFire;  //记录下一发子弹的时间
    public float fireRate;    //控制子弹发射的间隔时间
    public GameObject shot;    //保存我们发射的子弹Prefab
    public Transform shotSqawn;   //保存子弹的发射点

	void Update () {
        //按下发射键（ctrl或鼠标左键）并且到达下一发子弹发射时间
        if (Input.GetButton("Fire1") && Time.time > nextFire)
        {
            nextFire = Time.time + fireRate;//重置下一发子弹发射时间
           //Instantiate方法生成一发子弹
           //3个参数，一个是发射的对象(复制出一个新的)，一个是发射对象的position，一个是发射对象的rotation
            Instantiate(shot, shotSqawn.position, shotSqawn.rotation);
            GetComponent<AudioSource>().Play();
        }
    }
```

* **6.**回收子弹，飞船每发射一颗子弹，就会生成一个**Bolt**对象。创建一个盒子，这个盒子大小和我们的屏幕差不多，然后我们可以判断如何子弹飞出了这个盒子，就意味着子弹飞出了屏幕，这个时候我们就可以通知程序来回收子弹。创建**cube**，命名**Boundary**，将盒子调整到屏幕大小，删除 mesh renderer，添加Box Colider，is Trigger set on，为盒子添加脚本**DestroyByBoundary**

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1730128-f20ac08e15f7fe48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```csharp
     //DestroyByBoundary中添加碰撞的触发方法。即当子弹碰到时，子弹被回收
    void OnTriggerExit(Collider other)
    {
        //注意，避免将飞船回收
        if(other.tag != "Player")
        {
            Destroy(other.gameObject);
        }
    }
```

* **7.**创建障碍，创建GameObject，命名为**Asteroid**，为**Asteroid**添加美术效果，我们在**Assets->Models**文件夹中找到**prop_asteroid_01**文件，然后将其拖入**Asteroid**做其的子对象，为**Asteroid**添加Rigidibody，diselect use gravity，将阻力drag 和angular drag设为0，添加capsule collider，编写脚本**RandomRotator**，使其自动旋转

```csharp
    public float tumble;
    //tumble来设置障碍翻滚的幅度
	void Start () {
        GetComponent<Rigidbody>().angularVelocity = Random.insideUnitSphere * tumble;
    }

```

* **8.**首先我们需要检测子弹和障碍是否碰撞，如果碰撞了，我们需要创建一个爆炸效果来给予反馈，最后我们还需要将子弹和障碍删除掉。再为子弹编写一个脚本**DestroyByContact**
```csharp
    public GameObject explosion;    //爆炸效果
    public GameObject playerExplosion;
    void OnTriggerEnter(Collider other)
    {
        //初始时障碍物会与盒子boundary发生碰撞触发此函数，要规避此种触发
        if (other.tag == "Boundary")
        {
            return;
        }
        //生成一个爆炸效果
        Instantiate(explosion, transform.position, transform.rotation);
        if(other.tag == "Player")
        {
            Instantiate(playerExplosion, other.transform.position, other.transform.rotation);
            gameController.GameOver();
        }

        gameController.AddScore(scoreValue);
        //Debug.Log(other.name);
        Destroy(other.gameObject);
        Destroy(gameObject);
    }
```
回到Unity编辑器，接着在**Assets->Prefabs->VFX->Explosions**文件夹中找到**explosion_asteroid**文件，然后拖入到我们定义的**explosion**变量中。

* **8.** 使障碍物向下移动，随机生成障碍。mover是之前为子弹添加的脚本，同样可以用到障碍物上，只需将speed设置为负数，即可实现向下移动。新建一个GameObject， 命名为GameController，实现这些逻辑

```csharp
    public GameObject hazard;   //设置生成的障碍对象
    public Vector3 spawnValue;  //设置障碍生成的位置
    public int hazardCount;     //每次生成的障碍的数量
    public float spawnWait;     //每个障碍生成的间隔时间
    public float startWait;     //游戏开始生成障碍的准备时间
    public float waveWait;      //设置每一波的间隔时间


    IEnumerator SpawnWaves()
    {
        //让代码等待一定时间，这是个协同程序，可以让游戏不暂停的同时，让代码暂停
        yield return new WaitForSeconds(startWait);
        while (true)
        {
            for (int i = 0; i < hazardCount; i++)
            {
                //随机生成障碍位置
                Vector3 spawnPosition = new Vector3(Random.Range(-spawnValue.x, spawnValue.x), spawnValue.y, spawnValue.z);
                //Rotation属性如何确定，Rotation属性是一个Quaternion类型的变量，
                //所以我们定义了一个spawnRotation的变量，用来记录障碍的Rotation属性，在这里我们不需要让障碍一开始带着任何的旋转
                //所以我们直接赋值Quaternion.identity
                Quaternion spawnRotation = Quaternion.identity;

                Instantiate(hazard, spawnPosition, spawnRotation);

                yield return new WaitForSeconds(spawnWait);
            }
            yield return new WaitForSeconds(waveWait);

            if(gameOver == true)
            {
                restartText.text = "Press R for Restart";
                restart = true;
                break;
            }
        }

    }

    	void Start () {
            //因为SpawnWaves中有协同程序，StartCoroutine为启动一个协同程序
           StartCoroutine(SpawnWaves());
	}


```

* **9.** 回收爆炸效果，新建脚本**DestroyByTime**，lifetime为存活时间，将此脚本添加到perfab下的爆炸效果中，并设置lifetime。

```csharp
    public float lifeTime;

	// Use this for initialization
	void Start () {
        Destroy(gameObject,lifeTime);
	}
```

* **10.**添加背景音乐，**Assets->Audio**文件夹中找到背景音乐**music_background**直接拖入**GameController**对象，同理，障碍爆炸和飞船爆炸音效，也是将其音频文件直接拖到对应的perfab文件中，这些音频文件是默认play on awake的，其中还要讲背景音乐选中loop，子弹音效不能设为play on wake，只需要在发射时有声音，需要在脚本中设置，只需在**PlayController**中添加

```csharp
	void Update () {
        //按下发射键（ctrl或鼠标左键）并且到达下一发子弹发射时间
        if (Input.GetButton("Fire1") && Time.time > nextFire)
        {
            nextFire = Time.time + fireRate;
            //Instantiate方法生成一发子弹，他有3个参数
           //一个是发射的对象(复制出一个新的)，一个是发射对象的position，一个是发射对象的rotation
            Instantiate(shot, shotSqawn.position, shotSqawn.rotation);

            //当子弹发射时开启音效
            GetComponent<AudioSource>().Play();
        }
    }
```

* **11.**游戏计分按钮等，新建 scoreText，restartText，gameOverText，并调整其在屏幕上的位置。在GameController脚本设置。

![](http://upload-images.jianshu.io/upload_images/1730128-fd3a02a458003684.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```csharp
    public Text scoreText;
    public Text restartText;
    public Text gameOverText;
    private bool gameOver;
    private bool restart;
    private int score;

    IEnumerator SpawnWaves()
    {
        //让代码等待一定时间，这是个协同程序，可以让游戏不暂停的同时，让代码暂停
        yield return new WaitForSeconds(startWait);
        while (true)
        {
            for (int i = 0; i < hazardCount; i++)
            {
                //随机生成障碍位置
                Vector3 spawnPosition = new Vector3(Random.Range(-spawnValue.x, spawnValue.x), spawnValue.y, spawnValue.z);
                //Rotation属性如何确定，Rotation属性是一个Quaternion类型的变量，
                //所以我们定义了一个spawnRotation的变量，用来记录障碍的Rotation属性，在这里我们不需要让障碍一开始带着任何的旋转
                //所以我们直接赋值Quaternion.identity
                Quaternion spawnRotation = Quaternion.identity;

                Instantiate(hazard, spawnPosition, spawnRotation);

                yield return new WaitForSeconds(spawnWait);
            }
            yield return new WaitForSeconds(waveWait);

            if(gameOver == true)
            {
                restartText.text = "Press R for Restart";
                restart = true;
                break;
            }
        }

    }

    void UpdateScore()
    {
        scoreText.text = "Score :" + score;
    }

    public void  AddScore(int newScoreValue)
    {
        score += newScoreValue;
        UpdateScore();
    }

    public void GameOver ()
    {
        gameOverText.text = "Game Over !";
        gameOver = true;

    }
	void Start () {
        gameOver = false;
        restart = false;
        restartText.text = "";
        gameOverText.text = "";
        score = 0;
        UpdateScore();
        //因为SpawnWaves中有协同程序，StartCoroutine为启动一个协同程序
        StartCoroutine(SpawnWaves());
	}

	void Update () {
        if (restart)
        {
            if (Input.GetKeyDown(KeyCode.R))
            {
                //重新加载
                Application.LoadLevel(Application.loadedLevel);
            }
        }
	}
```
在GameController脚本中设置计分的相关方法，在DestoryByContact的脚本中调用这些方法
```csharp
using UnityEngine;
using System.Collections;

public class DestroyByContact : MonoBehaviour {

    public GameObject explosion;
    public GameObject playerExplosion;



    public int scoreValue;//分值
    private GameController gameController; //gameController脚本类

    // Use this for initialization
    void Start () {
        //获取GameController的实例
        //首先找到GameController的GameObject，然后在通过GetComponent方法来获取GameController脚本的实例
        GameObject gameControllerObject = GameObject.FindWithTag("GameController");
        if (gameControllerObject != null)
        {
            gameController = gameControllerObject.GetComponent<GameController>();
        }
        if (gameController == null)
        {
            Debug.Log("Cannot find 'GameController' script");
        }
    }

	// Update is called once per frame
	void Update () {

	}

    void OnTriggerEnter(Collider other)
    {
        //初始时障碍物会与盒子boundary发生碰撞触发此函数，要规避此种触发
        if (other.tag == "Boundary")
        {
            return;
        }
        //生成一个爆炸效果
        Instantiate(explosion, transform.position, transform.rotation);

        //飞船碰到障碍物时爆炸效果
        if(other.tag == "Player")
        {
            Instantiate(playerExplosion, other.transform.position, other.transform.rotation);

            //飞船爆炸时调用gameover方法
            gameController.GameOver();
        }

        gameController.AddScore(scoreValue);
        //Debug.Log(other.name);
        Destroy(other.gameObject);
        Destroy(gameObject);
    }
}

```


总结：在实际操作中，对unity的一些构建，基本的操作不熟悉，还有场景的设置即位置等。了解到了脚本的功能，控制GameObject，触发事件，不同的脚本之间相互调用。这个游戏，通过脚本新建的对象如子弹障碍物是不会自动回收的，需要在最外围设置一个框来检测物体碰撞触发事件进行回收，同时爆炸时Instantiate方法产生的爆炸效果，也不会被自动回收，需要对这些效果写脚本DestoryByTime定时进行Destory。

***

## 画一个非常丑的图。。。

![](http://upload-images.jianshu.io/upload_images/1730128-a678f218642875d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* PlayerController：飞船的移动，发射子弹。
* Mover：设置速度，自动控制移动。
* RandomRotator： 自动控制障碍物旋转。
* DestoryByContact：碰撞时触发的方法，物体消失，爆炸特效，计分，飞船爆炸时调用GameController的GameOver方法。
* DestoryByBoundary：子弹或障碍物出边界，即与Boundary相碰撞触发方法Destory对象。
* DestoryByTime：特效对象Destory。
* GameController： 产生障碍物，计分，Restart等方法实现。
