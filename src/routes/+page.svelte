<script>
    let text = "";
    let emotionScores = {};
    let recommendedTracks = [];
    let baseUrl = "http://127.0.0.1:5000";  // Put your backend address here

    function getEmotionColor(emotion) {
        const colors = {
            joy: 'yellow',
            sadness: 'blue',
            surprise: 'orange',
            anger: 'purple',
            love: 'red'
        };
        return colors[emotion] || 'grey';
    }

    async function analyzeEmotions() {
        if (text) {
            const emotionResponse = await fetch(
                `${baseUrl}/emotion/${encodeURIComponent(text)}`,
            );
            if (emotionResponse.ok) {
                emotionScores = await emotionResponse.json();
                await getMusicRecommendations();
            } else {
                console.error(
                    "Error fetching emotion scores:",
                    emotionResponse.statusText,
                );
            }
        }
    }

    function getHighestEmotion() {
        return Object.keys(emotionScores).reduce((a, b) => emotionScores[a] > emotionScores[b] ? a : b);
    }

    async function getMusicRecommendations() {
        let maxEmotion = getHighestEmotion();

        const musicResponse = await fetch(`${baseUrl}/music/${maxEmotion}`);
        if (musicResponse.ok) {
            recommendedTracks = await musicResponse.json();
        } else {
            console.error(
                "Error fetching music recommendations:",
                musicResponse.statusText,
            );
        }
    }
</script>

<div class="app">
    <div class="header">
        <h1>Listen to Your Mood</h1>
    </div>
    
    <div class="input-area">
        <input class="input-box"
            type="text"
            bind:value="{text}"
            placeholder="Type your text here..."
        />
        <button class="button" on:click="{analyzeEmotions}">Analyze Emotions</button>
    </div>
    
    {#if Object.keys(emotionScores).length > 0}
    <div class="emotion-scores">
        <h2>Your Emotion Scores</h2>
        <div class="scores">
            {#each Object.entries(emotionScores) as [emotion, score]}
            <span class={`emotion-tag ${getEmotionColor(emotion)}`}>
                <strong>{emotion.charAt(0).toUpperCase() + emotion.slice(1)}:</strong>
                {score}
            </span>
            {/each}
        </div>
    </div>
    {/if}
    
    {#if recommendedTracks.length > 0}
    <div class="recommended-tracks">
        <h2>Recommended Tracks</h2>
        <div class="tracks-container">
            {#each recommendedTracks as track}
            <div class="track-card">
                <img src={track.album_image} alt={`Album cover for ${track.name}`} class="album-cover">
                <div class="track-info">
                    <p class="track-name">{track.name}</p>
                    <p class="track-artist">{track.artist}</p>
                    <audio controls src="{track.preview_url}">
                        Your browser does not support the audio element.
                    </audio>
                    <a href="{track.spotify_url}" target="_blank">Listen on Spotify</a>
                </div>
            </div>
            {/each}
        </div>
    </div>
    {/if}
</div>

<style>
    /* rubik-latin-wght-normal */
    @font-face {
        font-family: 'Rubik Variable';
        font-style: normal;
        font-display: swap;
        font-weight: 300 700;
        src: url(https://cdn.jsdelivr.net/fontsource/fonts/rubik:vf@latest/latin-wght-normal.woff2) format('woff2-variations');
        unicode-range: U+0000-00FF,U+0131,U+0152-0153,U+02BB-02BC,U+02C6,U+02DA,U+02DC,U+0300-0301,U+0303-0304,U+0308-0309,U+0323,U+0329,U+2000-206F,U+2074,U+20AC,U+2122,U+2191,U+2193,U+2212,U+2215,U+FEFF,U+FFFD;
    }
    .app {
        font-family: 'Rubik Variable';
    }
    .header {
        text-align: center;
        margin-top: 20px;
    }
    .input-area {
        text-align: center;
        margin-top: 20px;
    }
    .input-box {
        width: 25%;
        padding: 4px;
        margin: 8px 0;
        border: 1px solid #ccc;
        border-radius: 6px;
    }
    .button {
        appearance: button;
        background-color: #1652F0;
        border: 1px solid #1652F0;
        border-radius: 6px;
        box-sizing: border-box;
        color: #FFFFFF;
        cursor: pointer;
        overflow: visible;
        padding: 4px 6px;
        position: relative;
        text-align: center;
        text-transform: none;
        transition: all 80ms ease-in-out;
        user-select: none;
        -webkit-user-select: none;
        touch-action: manipulation;
        width: fit-content;
    }
    .button:disabled {
        opacity: .5;
    }
    .button:focus {
        outline: 0;
    }
    .button:hover {
        background-color: #0A46E4;
        border-color: #0A46E4;
    }
    .button:active {
        background-color: #0039D7;
        border-color: #0039D7;
    }

    .emotion-scores {
        text-align: center;
        margin-top: 20px;
    }
    .scores {
        display: inline-flex;
        flex-wrap: wrap;
        justify-content: center;
        gap: 10px;
    }
    .emotion-tag {
        background-color: #f0f0f0;
        border-radius: 20px;
        padding: 5px 15px;
        font-size: 0.9em;
        margin: 5px;
    }
    .yellow { background-color: rgba(241, 196, 15, 0.5); }
    .blue { background-color: rgba(52, 152, 219, 0.5); }
    .purple { background-color: rgba(155, 89, 182, 0.5); }
    .orange { background-color: rgba(230, 126, 34, 0.5); }
    .red { background-color: rgba(231, 76, 60, 0.5); }
    .grey { background-color: rgba(127, 140, 141, 0.5); }

    .recommended-tracks {
        text-align: center;
        margin-top: 20px;
    }
    .tracks-container {
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
    }
    .track-card {
        margin: 10px;
        border: 1px solid #ddd;
        padding: 15px;
        border-radius: 8px;
        width: 300px;
        background-color: rgba(29, 185, 84, 0.5);
    }
    .album-cover {
        width: 100%;
        height: auto;
        border-radius: 4px;
    }
    .track-info {
        margin-top: 10px;
    }
    .track-name {
        font-weight: bold;
    }
    .track-artist {
        color: #555;
        margin-bottom: 10px;
    }
    audio {
        width: 100%;
        margin-bottom: 10px;
    }
    a {
        text-decoration: none;
    }
    a:hover {
        text-decoration: underline;
    }
</style>
