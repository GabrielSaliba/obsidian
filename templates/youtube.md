<%*
const url = await tp.system.prompt("Paste the YouTube link");

// Extract video ID
const idMatch = url.match(/(?:v=|youtu\.be\/)([a-zA-Z0-9_-]+)/);
const videoId = idMatch ? idMatch[1] : "";

if(!videoId){
  tR += "Invalid YouTube link";
} else {
  // Replace YOUR_API_KEY with your YouTube Data API key
  const apiKey = "AIzaSyD0Wv7jC42ta_2z70MiVsyk19Gl0bCOIjw";
  const apiUrl = `https://www.googleapis.com/youtube/v3/videos?part=snippet,contentDetails,statistics&id=${videoId}&key=${apiKey}`;
  
  const res = await fetch(apiUrl);
  const data = await res.json();
  
  if(data.items && data.items.length > 0){
    const video = data.items[0];
    const title = video.snippet.title;
    const channel = video.snippet.channelTitle;
    const durationISO = video.contentDetails.duration;
    const publishedAtRaw = video.snippet.publishedAt;

    const viewsCount = parseInt(video.statistics.viewCount);

    // Convert ISO 8601 duration to readable format
    const isoToMinutes = (iso) => {
      const match = iso.match(/PT(?:(\d+)H)?(?:(\d+)M)?(?:(\d+)S)?/);
      const h = parseInt(match[1]||0);
      const m = parseInt(match[2]||0);
      const s = parseInt(match[3]||0);
      return `${h>0?h+"h ":""}${m>0?m+"m ":""}${s}s`;
    }
    const duration = isoToMinutes(durationISO);

    // Format views to be readable (e.g. 1.2M, 135K)
    const formatViews = (num) => {
      if(num >= 1e9) return (num / 1e9).toFixed(1).replace(/\.0$/, '') + 'B';
      if(num >= 1e6) return (num / 1e6).toFixed(1).replace(/\.0$/, '') + 'M';
      if(num >= 1e3) return (num / 1e3).toFixed(1).replace(/\.0$/, '') + 'K';
      return num.toString();
    }
    const views = formatViews(viewsCount);

    // Format published date to "Month DD, YYYY"
    const formatDate = (dateStr) => {
      const date = new Date(dateStr);
      const options = { year: 'numeric', month: 'long', day: 'numeric' };
      return date.toLocaleDateString('en-US', options);
    }
    const publishedAt = formatDate(publishedAtRaw);

    // Build the card without thumbnail + 3 blank lines at the end
    tR += `> ## ${title}\n`;
    tR += `> **Channel:** ${channel}  \n`;
    tR += `> **Duration:** ${duration}  \n`;
    tR += `> **Published:** ${publishedAt}  \n`;
    tR += `> **Views:** ${views}  \n`;
    tR += `> **Link:** [YouTube Video](${url})\n\n`;
    tR += `![](https://www.youtube.com/watch?v=${videoId})\n\n\n`;
  } else {
    tR += "Video not found\n\n\n";
  }
}
%>

[[Library of Alexandria]]