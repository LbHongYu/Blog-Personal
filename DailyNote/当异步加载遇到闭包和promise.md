## 异步加载遇到闭包和promise


    这篇文章分享的是，用闭包去解决用循环数组发送请求，及其一些延伸问题。

涉及的主要知识点：
闭包，
Promise，
异步加载

需求：一个tabs的每一项后端数据会不定时的更新，每当更新的时候后，在用户进入页面后需要用一个”N” icon 表明这个tab有更新，用户点击这个tab会有一个弹窗提醒用户更新，用户可以接受或拒绝更新。用户拒绝更新的话，把这个 tabId发送给后端，告诉后端拒绝了这个tabId的更新。后端开发给了两个API，一个是取tab list， 一个是获取和更新tabId状态的(getStatus方法)。

下次重新进入这个页面的时候，在得到tab list数据之后，上次被拒绝的tabId 的 新版本标识依然存在tab list中，这些标识是不可用的，我们要提取所有有新版本表示的tabId，然后把这些tabId一一的与后端检查，如果检查到是被拒绝的更新，那么在显示tabs的时候，该tab就不需要显示”N” icon 了。

PS：关于tab新版的标识这里的说明做了处理。

示例图：
 ![tab.png](../picture//DailyNote/当异步加载遇到闭包和promise/tab.png "tab")

思考点：
1. 不定长度数组；
2. 异步加载；
3. 所有的请求返回后再渲染tab list。

看到前两个点，心里第一个反应就是，这是一个“不简单”的数组循环，如果不加思考的写出这个解决方案。

```
function getDiscardedTabId(allNewVersionTabId){
    let discardedVersion = [];
    if(/* is allNewVersionTabId a array*/){
        for(var i = 0; i < allNewVersionTabId.length; i++){
           API.getStatus(allNewVersionTabId[i]).then(res=>{
                discardedVersion.push(allNewVersionTabId[i]);
            },err=>{
                /*handle error message*/
            })
        }
    }else{
        console.warn("allNewVersionTabId should be a array");
    }
    retrun discardedVersion;
}

getDiscardedTabId(allNewVersionTabId);
```
这里会有一个问题：虽然数组中的每个tabId都可以发送一个属于自己请求，但是then的成功回调里面的i已经变成了allNewVersionTabId.length，allNewVersionTabId[allNewVersionTabId.length]当然就是undefined了，得不到你想要的那个tabId。

针对前两个思考点可以用以下两种方案解决，解决思路就是我们需要在每个tabId的请求成功回调保留索引值

1.
```
function getDiscardedTabId(allNewVersionTabId){
    let discardedVersion = [];
    if(/* is allNewVersionTabId a array*/){
        for(var i = 0; i < allNewVersionTabId.length; i++){
            (function(tabId){
                API.getStatus(tabId).then(res=>{								
                    discardedVersion.push(tabId);
                },err=>{
                    /*handle error message*/				
                })
            })(allNewVersionTabId[i])			
        }
    }else{
        console.warn("allNewVersionTabId should be a array");
    }

    retrun discardedVersion;
}
```
2.

```
function getDiscardedTabId(allNewVersionTabId){
let discardedVersion = [];
    if(/* is allNewVersionTabId a array*/){
        for(let i = 0; i < allNewVersionTabId.length; i++){
            API.getStatus(allNewVersionTabId[i]).then(res=>{			
                discardedVersion.push(allNewVersionTabId[i]);
            },err=>{
                /*handle error message*/				
            })
        }
    }else{
        console.warn("allNewVersionTabId should be a array");
    }
    retrun discardedVersion;
}
```
第一个方案是使用的是闭包，第二个使用的使用let声明i，闭包是为了保留每个循环的作用域，使用let的原理是，使每个循环的代码块，都是一个块作用域。

现在就是还有第三个思考点需要解决了，前面的代码得到的discardedVersion永远都是空数组，别问为什么，问就是异步延时！

固然使用promise. 我们把获取所有tabId状态当作一个任务，话不多说，直接上代码。
```
function getDiscardedTabId(allNewVersionTabId){
    let discardedVersion = [];
    if(/* is allNewVersionTabId a array*/){
        let promise = new Promise(function(resolve, reject) {
            for(let i = 0; i < allNewVersionTabId.length; i++){ 
                API.getStatus(allNewVersionTabId[i]).then(res=>{
                    if(/*this tabId has been discarded*/){
                        //将被拒绝的tabId添加到数组中，
                        //做作为promise的返回值
                        discardedVersion.push(allNewVersionTabId[i])
                    }
                    //当完成所有的请求，promise返回完成的状态
                    if(i === allNewVersionDashboard.length-1){
                        resolve(discardedVersion);
                    }
                },err=>{
                    
                    /*handle error message*/
                    
                    if(i === allNewVersionDashboard.length-1){
                        resolve(discardedVersion);
                    }
                })					
            }			
        })
        return promise;		
    }else{
        return Promise.resolve([]);
        console.warn("allNewVersionTabId should be a array");
    }
}
```

写到这里的时候，我以为自己的已经大功告成，但是经过一番测试发现，每次出现的“N”数量不同，总是小于准确的数量。导致的原因就是加载延时，比如有五个请求，最后一个请求已经返回，而前面的请求还没有返回，if(i === allNewVersionDashboard.length-1)的判断就会让前面没有返回的TabId丢失了，所以我们需要确保每一个tabId的请求都返回了。我们添加一个变量，每返回一个请求，这个变量就加1。当加到数组的长度的时候，说明所有的请求都返回了。

```
function getDiscardedTabId(allNewVersionTabId){
    let discardedVersion = [],
    idIndex = 0;

    if(/* is allNewVersionTabId a array*/){
        let promise = new Promise(function(resolve, reject) {
            for(let i = 0; i < allNewVersionTabId.length; i++){
                API.getStatus(allNewVersionTabId[i]).then(res=>{
                    //每完成一个请求，标识inIndex加一
                    //确保每一个tabId的请求都返回
                    idIndex++; 
                    if(/*this tabId has been discarded*/){
                        discardedVersion.push(allNewVersionTabId[i])
                    }
                    if(idIndex === allNewVersionDashboard.length-1){
                        resolve(discardedVersion);
                    }
                },err=>{
                    /*handle error message*/	
                    idIndex++;
                    if(idIndex === allNewVersionDashboard.length-1){
                        resolve(discardedVersion);
                    }
                })					
            }			
        })
        return promise;
    }else{
        return Promise.resolve([]);
        console.warn("allNewVersionTabId should be a array");
    }
}

getDiscardedTabId(allNewVersionTabId).then(res=>{
    let discardedTabId = res;
    /*show tab list*/
});
```
现在是真正的大功告成了。

