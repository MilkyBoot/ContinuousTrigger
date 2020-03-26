![](https://github.com/iostyle/ImageRepo/blob/master/icon_continuous_trigger.png)
# ContinuousTrigger [![](https://jitpack.io/v/iostyle/ContinuousTrigger.svg)](https://jitpack.io/#iostyle/ContinuousTrigger)
用于按序执行一系列任务，可随时绑定（如接口返回），可对每个步骤设置超时响应时间。 

可以将项目clone下来，里面有demo可以修改及测试，喜欢请Star⭐️

Step 1. Add it in your root build.gradle at the end of repositories:

	allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
Step 2. Add the dependency

	dependencies {
		implementation 'com.github.iostyle:ContinuousTrigger:1.0.2'
	}
  
## Method
|属性/方法|描述|
|--|--|
|id|唯一标识符，用来注册/绑定触发器|
|timeout|超时时间，当前节点超时时间内没有attach将跳转到下一节点，默认值为-1|
|with|Builder模式构造注册|
|create|Builder模式创建实例并初始化|
|register|实例化方式有序注册|
|attach|根据ID绑定触发器|
|next|下一步|
|cancel|根据ID取消对应触发器，如果是当前阻塞则自动执行下一个|
|response|响应并关闭超时线程|
|clear|清空|

## Example
```
      trigger = ContinuousTrigger.Builder()
                .with(
                        Trigger().also {
                            it.id = "test1"
                            it.timeout = 2000
                        }
                )
                .with(
                        Trigger().also {
                            it.id = "test2"
                            it.timeout = 2000
                        }
                )
                .with(
                        Trigger().also { it.id = "test3" }
                ).create()
      
      //或者直接实例化使用
      trigger = ContinuousTrigger()
                .register(
                        Trigger().also {
                            it.id = "test1"
                            it.timeout = 2000
                        })
                .register(
                        Trigger().also {
                            it.id = "test2"
                            it.timeout = 2000
                        })
                .register(
                        Trigger().also {
                            it.id = "test3"
                        })
              
//这里举一个按序弹窗的例子，也许这些弹窗所需数据来自不同的接口，你可以在任何位置任何时候attach，触发器会按注册顺序执行
       trigger?.attach("test3", object : Trigger.Strike {
            override fun strike() {
                trigger?.response()
                AlertDialog.Builder(this@MainActivity).setMessage("test3").setOnDismissListener {
                    trigger?.next()
                }.show()
            }
        })           
       
       GlobalScope.launch {
            delay(1500)
            withContext(Dispatchers.Main) {
                trigger?.attach("test1", object : Trigger.Strike {
                    override fun strike() {
                        trigger?.response()
                        AlertDialog.Builder(this@MainActivity).setMessage("test1").setOnDismissListener {
                            trigger?.next()
                        }.show()
                    }
                })
            }
        }


        GlobalScope.launch {
            delay(6000)
            withContext(Dispatchers.Main) {
                trigger?.attach("test2", object : Trigger.Strike {
                    override fun strike() {
                        trigger?.response()
                        AlertDialog.Builder(this@MainActivity).setMessage("test2").setOnDismissListener {
                            trigger?.next()
                        }.show()
                    }
                })
            }
        }

```
