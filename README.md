 <!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>极简全自动影院</title>
    <!-- 核心配置1：破解图片防盗链，确保海报100%显示 -->
    <meta name="referrer" content="no-referrer">
    <style>
        * { box-sizing: border-box; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; background: #141414; color: #fff; margin: 0; padding: 0; }
        header { background: #000; padding: 15px 20px; position: sticky; top: 0; z-index: 100; box-shadow: 0 4px 10px rgba(0,0,0,0.5); }
        .nav-container { max-width: 1200px; margin: 0 auto; display: flex; flex-wrap: wrap; justify-content: space-between; align-items: center; gap: 15px; }
        .logo { font-size: 24px; font-weight: bold; color: #e50914; cursor: pointer; text-decoration: none; }
        .nav-tabs { display: flex; gap: 10px; }
        .tab-btn { background: #2f2f2f; color: #fff; border: none; padding: 8px 16px; border-radius: 4px; cursor: pointer; font-size: 14px; transition: 0.2s; }
        .tab-btn.active, .tab-btn:hover { background: #e50914; }
        .search-box { display: flex; gap: 5px; }
        .search-box input { background: #2f2f2f; border: 1px solid #444; color: #fff; padding: 8px 12px; border-radius: 4px; outline: none; width: 200px; }
        .search-box button { background: #e50914; color: #fff; border: none; padding: 8px 16px; border-radius: 4px; cursor: pointer; }
        
        main { max-width: 1200px; margin: 20px auto; padding: 0 20px; }
        
        /* 播放器区域 */
        #player-wrapper { display: none; background: #000; border-radius: 8px; overflow: hidden; margin-bottom: 30px; box-shadow: 0 10px 30px rgba(0,0,0,0.7); }
        .player-header { background: #1f1f1f; padding: 12px 20px; display: flex; justify-content: space-between; align-items: center; }
        .player-header h3 { margin: 0; font-size: 16px; color: #e50914; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
        .close-btn { background: none; border: none; color: #aaa; font-size: 20px; cursor: pointer; }
        .close-btn:hover { color: #fff; }
        .iframe-container { position: relative; width: 100%; padding-top: 56.25%; /* 16:9 比例 */ }
        .iframe-container iframe { position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none; }
        
        /* 电影网格列表 */
        #movie-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(140px, 1fr)); gap: 20px; }
        .movie-card { background: #1f1f1f; border-radius: 6px; overflow: hidden; cursor: pointer; transition: transform 0.2s, box-shadow 0.2s; position: relative; }
        .movie-card:hover { transform: translateY(-5px); box-shadow: 0 5px 15px rgba(0,0,0,0.5); }
        .poster-wp { width: 100%; padding-top: 140%; position: relative; background: #2f2f2f; }
        .poster-wp img { position: absolute; top: 0; left: 0; width: 100%; height: 100%; object-fit: cover; }
        .note { position: absolute; top: 5px; right: 5px; background: rgba(229,9,20,0.85); font-size: 11px; padding: 2px 6px; border-radius: 3px; }
        .info { padding: 10px; }
        .info p { margin: 0; font-size: 13px; font-weight: bold; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
        
        .status-msg { text-align: center; padding: 5px 0; font-size: 16px; color: #aaa; width: 100%; grid-column: 1 / -1; }
    </style>
</head>
<body>

    <header>
        <div class="nav-container">
            <a class="logo" onclick="changeCategory('')">⚡ 极速全自动影院</a>
            <div class="nav-tabs">
                <button class="tab-btn active" onclick="changeCategory('')">最新</button>
                <button class="tab-btn" onclick="changeCategory('1')">电影</button>
                <button class="tab-btn" onclick="changeCategory('2')">连续剧</button>
                <button class="tab-btn" onclick="changeCategory('3')">综艺</button>
                <button class="tab-btn" onclick="changeCategory('4')">动漫</button>
            </div>
            <div class="search-box">
                <input type="text" id="search-input" placeholder="输入你想看的片名..." onkeydown="if(event.keyCode==13) searchMovies()">
                <button onclick="searchMovies()">搜索</button>
            </div>
        </div>
    </header>

    <main>
        <!-- 核心配置2：极简无广告解析播放器 -->
        <div id="player-wrapper">
            <div class="player-header">
                <h3 id="player-title">正在加载...</h3>
                <button class="close-btn" onclick="closePlayer()">✕ 关闭播放</button>
            </div>
            <div class="iframe-container">
                <iframe id="main-player" src="" allowfullscreen></iframe>
            </div>
        </div>

        <!-- 电影列表 -->
        <div id="movie-grid">
            <div id="status-text" class="status-msg">正在全自动同步全网最新资源...</div>
        </div>
    </main>

    <script>
        // 核心配置3：已修复的完整量资源API
        const BASE_API = 'https://lziapi.com';
        // 核心配置4：已修复的完整XMLF播放器
        const PLAYER_API = 'https://xmflv.cc';

        let currentCategory = '';
        let searchQuery = '';

        async function loadMovies() {
            const grid = document.getElementById('movie-grid');
            const status = document.getElementById('status-text');
            status.style.display = 'block';
            status.innerText = '正在全自动同步全网最新资源...';
            
            const cards = grid.querySelectorAll('.movie-card');
            cards.forEach(c => c.remove());

            let targetUrl = BASE_API;
            if (searchQuery) {
                targetUrl += `&wd=${encodeURIComponent(searchQuery)}`;
            } else if (currentCategory) {
                targetUrl += `&t=${currentCategory}`;
            }

            try {
                const response = await fetch(targetUrl);
                const data = await response.json();
                status.style.display = 'none';
                
                if (!data.list || data.list.length === 0) {
                    status.style.display = 'block';
                    status.innerText = '未找到相关影视资源，换个关键词试试吧！';
                    return;
                }
                renderGrid(data.list);
            } catch (error) {
                status.style.display = 'block';
                status.innerText = '同步失败，请检查采集站接口是否在线。';
                console.error(error);
            }
        }

        function renderGrid(movies) {
            const grid = document.getElementById('movie-grid');
            
            movies.forEach(movie => {
                let playUrl = '';
                if (movie.vod_play_url) {
                    const lines = movie.vod_play_url.split('$$$');
                    const episodes = lines[0].split('#');
                    const parts = episodes[0].split('$');
                    playUrl = parts.length > 1 ? parts[1] : parts[0];
                }

                const card = document.createElement('div');
                card.className = 'movie-card';
                card.innerHTML = `
                    <div class="poster-wp">
                        <img src="${movie.vod_pic}" onerror="this.src='https://placeholder.com'">
                        <div class="note">${movie.vod_remarks || '高清'}</div>
                    </div>
                    <div class="info">
                        <p title="${movie.vod_name}">${movie.vod_name}</p>
                    </div>
                `;

                card.onclick = () => {
                    if (!playUrl || !playUrl.startsWith('http')) {
                        alert('该影片暂无有效播放源，请尝试其他影片');
                        return;
                    }
                    window.scrollTo({ top: 0, behavior: 'smooth' });
                    document.getElementById('player-wrapper').style.display = 'block';
                    document.getElementById('player-title').innerText = `正在播放：${movie.vod_name} (${movie.vod_remarks || '正片'})`;
                    document.getElementById('main-player').src = PLAYER_API + encodeURIComponent(playUrl.trim());
                };

                grid.appendChild(card);
            });
        }

        function changeCategory(catId) {
            currentCategory = catId;
            searchQuery = '';
            document.getElementById('search-input').value = '';
            
            const buttons = document.querySelectorAll('.tab-btn');
            buttons.forEach((btn, idx) => {
                btn.classList.remove('active');
                if((catId === '' && idx === 0) || (catId === '1' && idx === 1) || (catId === '2' && idx === 2) || (catId === '3' && idx === 3) || (catId === '4' && idx === 4)) {
                    btn.classList.add('active');
                }
            });
            loadMovies();
        }

        function searchMovies() {
            const input = document.getElementById('search-input').value.trim();
            if (!input) {
                alert('请输入搜索关键词');
                return;
            }
            searchQuery = input;
            currentCategory = '';
            document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
            loadMovies();
        }

        function closePlayer() {
            document.getElementById('player-wrapper').style.display = 'none';
            document.getElementById('main-player').src = '';
        }

        loadMovies();
    </script>
</body>
</html>
