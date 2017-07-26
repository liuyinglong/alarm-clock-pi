### node 控制 树莓派做的天气闹钟

在成都上班，下雨天堵车，迟到的概率会很大。  
正好手上有一块树莓派 ，做了一个晴雨闹钟。  
下雨天 早上 7:00叫我起床
晴天 早上 7:30叫我起床  
将自己喜欢的歌曲放在一个文件夹中，随机播放，防止听腻

以下是代码


```js
const UID = "U785B76FC9"; // 测试用 用户ID，请更换成您自己的用户ID
const KEY = "4r9bergjetiv1tsd"; // 测试用 key，请更换成您自己的 Key
let LOCATION = "双流"; // 除拼音外，还可以使用 v3 id、汉语等形式
let Api = require('./lib/api.js');
let api = new Api(UID, KEY);
let rainRegexp = /雨/;
let exec = require('child_process').exec;

let fs = require("fs");
let schedule = require("node-schedule");


function alarmClockTime(isRain) {
    let today = new Date();
    let Y = today.getFullYear();
    let M = today.getMonth();
    let D = today.getDate();
    let week = today.getDay();
    if (week === 0 || week === 6) {
        return new Date(Y, M, D, 9, 30, 0);
    }
    if (isRain) {
        return new Date(Y, M, D, 7, 0, 0);
    }
    return new Date(Y, M, D, 7, 30, 0);
}

//调用音乐
function playMusic() {
    let shellStr;
    let musicArray;
    try {
        musicArray = fs.readdirSync("/media/hd/music/alarm/");
        let fileNmae=musicArray[Math.floor(musicArray.length * Math.random())].replace(/ /g,"\\ ");
        shellStr = "mplayer /media/hd/music/alarm/" +fileNmae;
    } catch (err) {
        shellStr = "mplayer /media/hd/music/qiyue.mp3";
    }

    console.log(shellStr);

    exec(shellStr, function (err, data) {
        if (err) {
            console.log(err);
            return;
        }
        console.log("播放完成");
    })
}

function setAlarmClock(time) {
    schedule.scheduleJob(time, function () {
        playMusic();
    });
}

/**
 * 获取天气信息
 */
function getWetherInfo() {
    let getNowWeather = api.getWeather("/weather/now.json", {
        location: LOCATION
    });

    let getNextWeather = api.getWeather("/weather/daily.json", {
        location: LOCATION
    });

    Promise.all([getNowWeather, getNextWeather]).then(function (data) {
        let nowWeatherText = data[0].results[0].now.text;
        let dayWeatherText = data[1].results[0].daily[0].text_day;
        console.log(new Date() + " ：" + dayWeatherText);
        if (rainRegexp.test(nowWeatherText) || rainRegexp.test(dayWeatherText)) {
            //当天有雨 提前设置闹钟
            setAlarmClock(alarmClockTime(true));
        } else {
            //没有雨 延后设置闹钟
            setAlarmClock(alarmClockTime());
        }
    }).catch(function (err) {
        console.log(err);
        //如果保存
        setAlarmClock(alarmClockTime(true));
    });
}


schedule.scheduleJob("0 50 6 * * *", function () {
    getWetherInfo();
});


```