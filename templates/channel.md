<%*
const url = await tp.system.prompt("Paste the YouTube channel link");

// Extract handle or channel ID
let handleMatch = url.match(/youtube\.com\/@([a-zA-Z0-9._-]+)/);
let idMatch = url.match(/youtube\.com\/channel\/([a-zA-Z0-9_-]+)/);

let handle = handleMatch ? handleMatch[1] : null;
let channelId = idMatch ? idMatch[1] : null;

const apiKey = "AIzaSyD0Wv7jC42ta_2z70MiVsyk19Gl0bCOIjw";

let apiUrl;

if(channelId){
  apiUrl = `https://www.googleapis.com/youtube/v3/channels?part=snippet,statistics&id=${channelId}&key=${apiKey}`;
}
else if(handle){
  apiUrl = `https://www.googleapis.com/youtube/v3/channels?part=snippet,statistics&forHandle=${handle}&key=${apiKey}`;
}
else{
  tR += "Invalid YouTube channel link";
}

if(apiUrl){

  const res = await fetch(apiUrl);
  const data = await res.json();

  if(data.items && data.items.length > 0){

    const channel = data.items[0];

    const name = channel.snippet.title;
    const thumbnail = channel.snippet.thumbnails.high.url;
    const subs = parseInt(channel.statistics.subscriberCount);

    const formatSubs = (num) => {
      if(num >= 1e9) return (num / 1e9).toFixed(1).replace(/\.0$/, '') + 'B';
      if(num >= 1e6) return (num / 1e6).toFixed(1).replace(/\.0$/, '') + 'M';
      if(num >= 1e3) return (num / 1e3).toFixed(1).replace(/\.0$/, '') + 'K';
      return num.toString();
    }

    const subscribers = formatSubs(subs);

tR += `
<div class="yt-channel-card">

<img src="${thumbnail}" class="yt-channel-avatar">

<div class="yt-channel-info">

<strong>${name}</strong><br>
Subscribers: ${subscribers}<br>
<a href="${url}">Open Channel</a>

</div>
</div>

<br>
`;

  } else {
    tR += "Channel not found";
  }
}
%>

[[Library of Alexandria]]