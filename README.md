<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <title>禮包選擇器</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      display: flex;
      height: 100vh;
      margin: 0;
    }
    .left, .right {
      padding: 10px;
      overflow-y: auto;
    }
    .left {
      flex: 1;
      border-right: 1px solid #ccc;
    }
    .right {
      width: 500px;
      flex-shrink: 0;
    }
    .category {
      margin-bottom: 20px;
    }
    .category h3 {
      margin-bottom: 10px;
    }
    .buttons {
      display: grid;
      gap: 5px;
    }
    .season .buttons {
      grid-template-columns: repeat(6, 1fr);
    }
    .special .buttons, .event .buttons {
      grid-template-columns: repeat(4, 1fr);
    }
    button.selectable {
      padding: 8px;
      font-size: 14px;
      cursor: pointer;
    }
    button.selected {
      background-color: #add8e6;
    }
    textarea {
      width: 100%;
      height: 500px;
      font-size: 14px;
      white-space: pre-wrap;
    }
    .info {
      margin-top: 10px;
    }
  </style>
</head>
<body>

<div class="left">
  <div class="category season">
    <h3>季節</h3>
    <div class="buttons" id="season-buttons"></div>
  </div>
  <div class="category special">
    <h3>特殊禮包</h3>
    <div class="buttons" id="special-buttons"></div>
  </div>
  <div class="category event">
    <h3>活動禮包</h3>
    <div class="buttons" id="event-buttons"></div>
  </div>
</div>

<div class="right">
  <textarea id="result" readonly></textarea>
  <button onclick="copyText()">全選複製</button>
  <div class="info">
    特殊禮包數量：<span id="special-count">0</span><br>
    活動禮包數量：<span id="event-count">0</span>
  </div>
</div>

<script>
const templateText = `✎ 腿毛整理(՞• •՞)
«» （）
價格 ⨯ ᴘʀɪᴄᴇ 
𝐍𝐓𝐃 𝐑𝐌𝐁 𝐇𝐊𝐃 𝐌𝐘𝐁
───────── ─────────
◜季節◞ ⨯ ꜱᴇᴀꜱᴏɴ
｜｜｜½⅓⅔¼¾ ⁰
◜地圖◞ ⨯ ᴍᴀᴘ
晨島｜雲野｜雨林｜霞谷｜墓土｜禁閣
◜綁定◞ ⨯ ʟɪɴᴋ
║║║
𝖦𝖢 𝖦𝖦 𝖥𝖡 𝖭𝖲 𝖨𝖣 𝖧𝖶 𝖲𝖳 𝖯𝖲 𝖳𝖶
◜售後◞ ⨯ ɢᴜᴀʀᴀɴᴛᴇᴇ  
✔︎／✘
───────── ─────────
◜禮包◞ ⨯ ɢɪꜰᴛ
𖥨特殊› 
˳˳˳˳
𖥨活動› 
˳˳˳˳
◜退件◞ ⨯ ʀᴇᴛᴜʀɴ
˳˳˳˳
◜徽章◞ ⨯ ʙᴀᴅɢᴇ
𖥨實體›
˳˳˳˳
𖥨數據› 
˳˳˳˳
◜備註◞ ⨯ ᴏᴛʜᴇʀ
🕯️∣ 🤍 ∣ 🧨
───────── ─────────
文案皆為手工排版勿盜
實際物品皆以影片為主
文案需經腿毛本人授權後使用`;

const seasonItems = ["感恩","追光","歸屬","音韻","魔法","聖島","預言","夢想","重組","王子","風行","深海","表演","破曉","極光","緬懷","夜行","拾光","歸巢","色鹿","築巢","協奏","姆明","彩染","暮星"];
const specialItems = ["治癒白花","五周年斗","六周年斗","耳狗髮型","耳狗頭飾","耳狗斗篷","耳狗玩偶","絆愛套裝","愛麗絲裙","愛麗絲門","王子圍巾","星球夾克","狐狸玩偶","遠行長褲","摯愛裝扮","極光白鞋","遠行髮型","星月頭飾","極光翅膀","奉獻斗篷","人聲樂器","色鹿頭角","蓮花斗篷","史迪奇帽","史迪奇袍","樹精肩飾","姆明耳尾","妮妮斗篷","暮星白花","暗黑斗篷","海牛頭飾","海牛玩偶","海牛套裝","冥龍套裝"];
const eventItems = ["雛鳥之琴","凱旋手蝶","白韻吉他","週年吉他","高音提琴","竪式鋼琴","辦公藍斗","初始禮包","風之旅人","超凡旅人","林克套裝","飛蛾套裝","麻雀套裝","兔子棉褲","龍鱗長袍","絨帽髮型","橘子頭飾","小魚頭飾","龍鱗耳墜","盤龍圍巾","祥雲紙傘","福瑞紙扇","鯉魚套裝","福娃套裝","福蛇套裝","尋寶套裝","白色領結","情人馬尾","愛心髮箍","流星斗篷","愛心煙花","情人鞦韆","小蹺蹺板","情人小船","園丁裝扮","玫瑰披肩","荷葉綠傘","雙人茶几","三人茶桌","野餐地墊","海洋面具","海洋墨鏡","海螺項鍊","海龜肩飾","水漾髮型","綠芽斗篷","海洋斗篷","海龜斗篷","海浪斗篷","波浪套裝","海螺擺飾","彩虹束褲","彩虹跑鞋","彩虹面具","彩虹毛帽","彩虹單花","彩虹雙花","彩虹耳釘","彩虹耳機","彩鱗耳墜","彩虹泡泡","小狗頭飾","小狗玩偶","週年套餐","夏日涼鞋","水母肩飾","貝殼頭飾","陽光耳環","夏日披風","夏日陽傘","篝火點心","衝浪滑板","玉兔頭飾","手提燈籠","月華套裝","月光耳環","披肩外袍","競標頭飾","音律長褲","闊腿牛仔","紳士套裝","兔子拖鞋","火焰墨鏡","愛心墨鏡","炸毛領帶","伯爵面具","南瓜髮型","巫師髮型","蜘蛛龐克","枯枝頭角","蝙蝠斗篷","蜘蛛斗篷","蛛絲斗篷","伯爵披風","烏鴉斗篷","飛行掃帚","南瓜擺飾","炸毛貓貓","女巫套裝","貓咪套裝","雪人毛靴","聖誕毛帽","雪花頭飾","馴鹿頭角","瑞雪斗篷","白絨斗篷","雪怪斗篷","暖冬斗篷","雪花燈球","球門套組"];

let selected = {season: [], special: [], event: []};

function createButtons(items, containerId, category) {
  const container = document.getElementById(containerId);
  items.forEach(item => {
    const btn = document.createElement("button");
    btn.className = "selectable";
    btn.textContent = item;
    btn.onclick = () => {
      const index = selected[category].indexOf(item);
      if (index > -1) {
        selected[category].splice(index, 1);
        btn.classList.remove("selected");
      } else {
        selected[category].push(item);
        btn.classList.add("selected");
      }
      updateResult();
    };
    container.appendChild(btn);
  });
}

function group(items, groupSize = 4, delimiter = "˳") {
  const groups = [];
  for (let i = 0; i < items.length; i += groupSize) {
    groups.push(items.slice(i, i + groupSize).join(delimiter));
  }
  return groups.join("\n");
}

function updateResult() {
  const sortByOriginal = (selectedList, referenceList) =>
    referenceList.filter(item => selectedList.includes(item));

  const season = sortByOriginal(selected.season, seasonItems).join("｜");
  const special = group(sortByOriginal(selected.special, specialItems));
  const event = group(sortByOriginal(selected.event, eventItems));

  const text = templateText.replace("｜｜｜", season)
                           .replace("˳˳˳˳", special)
                           .replace("˳˳˳˳", event);

  document.getElementById("result").value = text;
  document.getElementById("special-count").textContent = selected.special.length;
  document.getElementById("event-count").textContent = selected.event.length;
}

function copyText() {
  const textarea = document.getElementById("result");
  textarea.select();
  document.execCommand("copy");
  alert("內容已複製到剪貼簿！");
}

createButtons(seasonItems, "season-buttons", "season");
createButtons(specialItems, "special-buttons", "special");
createButtons(eventItems, "event-buttons", "event");
</script>

</body>
</html>
