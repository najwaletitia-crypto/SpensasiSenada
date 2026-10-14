# SpensasiSenada
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="referrer" content="strict-origin-when-cross-origin" />
    <title>Daily Motivation</title>
    
    <!-- Firebase SDK -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
        import { getDatabase, ref, push, onValue, remove } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-database.js";
        
        window.firebaseFunctions = { initializeApp, getDatabase, ref, push, onValue, remove };
        window.onFirebaseLoaded();
    </script>

    <style>
        :root {
            --bg-color: #EFEBE9;
            --card-bg: #FFFFFF;
            --primary-brown: #5D4037;
            --secondary-brown: #8D6E63;
            --accent-gold: #D7CCC8;
            --text-dark: #3E2723;
        }
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', sans-serif; }
        body { background-color: var(--bg-color); color: var(--text-dark); line-height: 1.6; min-height: 100vh; display: flex; flex-direction: column; align-items: center; }
        header { background-color: var(--primary-brown); width: 100%; padding: 2rem 1rem; text-align: center; color: white; box-shadow: 0 4px 6px rgba(0,0,0,0.1); margin-bottom: 2rem; }
        h1 { font-size: 2.5rem; font-weight: 300; letter-spacing: 2px; }
        .container { width: 90%; max-width: 1000px; padding-bottom: 4rem; }
        
        /* Form Styles */
        .input-section { background: var(--card-bg); padding: 2rem; border-radius: 15px; box-shadow: 0 4px 15px rgba(93, 64, 55, 0.1); margin-bottom: 2rem; border: 1px solid var(--accent-gold); }
        .input-group { margin-bottom: 1rem; }
        label { display: block; margin-bottom: 0.5rem; font-weight: 600; color: var(--secondary-brown); }
        input, textarea { width: 100%; padding: 12px; border: 2px solid var(--accent-gold); border-radius: 8px; background: #FAFAFA; font-size: 1rem; outline: none; transition: 0.3s; }
        input:focus, textarea:focus { border-color: var(--primary-brown); }
        .hint { font-size: 0.8rem; color: #888; margin-top: 4px; }
        button.btn-submit { background-color: var(--primary-brown); color: white; padding: 12px 24px; border: none; border-radius: 8px; cursor: pointer; font-size: 1rem; width: 100%; transition: 0.3s; }
        button.btn-submit:hover { background-color: #3E2723; transform: translateY(-2px); }
        
        /* Admin & Feed */
        .admin-controls { text-align: right; margin-bottom: 1rem; }
        .btn-link { background: none; border: none; color: var(--secondary-brown); cursor: pointer; text-decoration: underline; }
        .feed-title { margin-bottom: 1rem; color: var(--primary-brown); border-bottom: 2px solid var(--accent-gold); padding-bottom: 10px; }
        .grid-posts { display: grid; grid-template-columns: repeat(auto-fill, minmax(320px, 1fr)); gap: 20px; }
        
        /* Card Post */
        .card { background: var(--card-bg); padding: 1.5rem; border-radius: 12px; box-shadow: 0 2px 8px rgba(93, 64, 55, 0.08); transition: 0.3s; border-top: 4px solid var(--secondary-brown); display: flex; flex-direction: column; }
        .card:hover { transform: translateY(-5px); box-shadow: 0 8px 20px rgba(93, 64, 55, 0.15); }
        .quote-text { font-size: 1.1rem; font-style: italic; margin-bottom: 1rem; flex-grow: 1; }
        
        /* Music Player */
        .music-player { width: 100%; margin-top: 10px; border-radius: 8px; overflow: hidden; background: #000; }
        .music-player iframe { width: 100%; height: 180px; border: none; }
        
        /* Text Only Song Info */
        .song-text { background-color: var(--bg-color); padding: 10px; border-radius: 8px; font-size: 0.9rem; color: var(--secondary-brown); display: flex; align-items: center; gap: 10px; margin-top: 10px; }
        
        .btn-delete { background-color: #ef5350; color: white; border: none; padding: 8px; border-radius: 5px; cursor: pointer; margin-top: 1rem; width: 100%; display: none; }
        .btn-delete:hover { background-color: #c62828; }
        
        /* Toast */
        #toast { visibility: hidden; min-width: 250px; background-color: #333; color: #fff; text-align: center; border-radius: 4px; padding: 16px; position: fixed; left: 50%; bottom: 30px; transform: translateX(-50%); z-index: 100; }
        #toast.show { visibility: visible; animation: fadein 0.5s, fadeout 0.5s 2.5s; }
        @keyframes fadein { from {bottom: 0; opacity: 0;} to {bottom: 30px; opacity: 1;} }
        @keyframes fadeout { from {bottom: 30px; opacity: 1;} to {bottom: 0; opacity: 0;} }
    </style>
</head>
<body>

    <header>
        <h1>DAILY MOTIVATION</h1>
        <p class="subtitle">Semangatkan hari dengan kata-kata dan musik</p>
        <p id="status" style="font-size:0.8rem; margin-top:5px; opacity:0.7;">Connecting...</p>
    </header>

    <div class="container">
        <div class="admin-controls">
            <button id="loginAdminBtn" class="btn-link" onclick="toggleAdminMode()">Login Admin</button>
        </div>

        <div class="input-section">
            <div class="input-group">
                <label>Kutipan Motivasi</label>
                <textarea id="quoteInput" rows="3" placeholder="Contoh: Hanya mereka yang mau mencoba yang akan berhasil..."></textarea>
            </div>
            <div class="input-group">
                <label>Rekomendasi Lagu (Link YouTube / Judul)</label>
                <input type="text" id="songInput" placeholder="Contoh: https://www.youtube.com/watch?v=... atau Judul Lagu">
                <div class="hint">💡 Paste link YouTube untuk memutar langsung!</div>
            </div>
            <button class="btn-submit" onclick="addPost()">Bagikan Motivasi</button>
        </div>

        <h2 class="feed-title">Kontribusi Terbaru</h2>
        <div class="grid-posts" id="postsContainer"></div>
    </div>

    <div id="toast"></div>

    <script>
        // --- ISI FIREBASE CONFIG ANDA ---
const firebaseConfig = {
  apiKey: "AIzaSyDL5ZABX2xRVW7uHfMScojndmf6hFdp7io",
  authDomain: "daily-motivation-8eb15.firebaseapp.com",
  databaseURL: "https://daily-motivation-8eb15-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId: "daily-motivation-8eb15",
  storageBucket: "daily-motivation-8eb15.firebasestorage.app",
  messagingSenderId: "914606940698",
  appId: "1:914606940698:web:87e23dcfc31c1975aaa7a3"
};
        // --------------------------------

        const ADMIN_PASS = "admin123";
        let isAdmin = false;
        let db;
        let postsRef;

        window.onFirebaseLoaded = function() {
            const app = window.firebaseFunctions.initializeApp(firebaseConfig);
            db = window.firebaseFunctions.getDatabase(app);
            postsRef = window.firebaseFunctions.ref(db, 'posts');
            
            document.getElementById('status').innerText = "🟢 Terhubung";
            
            window.firebaseFunctions.onValue(postsRef, (snapshot) => {
                renderPosts(snapshot.val());
            });
        };

        // Fungsi Ambil ID YouTube dari link
        function getYouTubeID(url) {
            if(!url) return null;
            const regExp = /^.*(youtu.be\/|v\/|u\/\w\/|embed\/|watch\?v=|&v=)([^#&?]*).*/;
            const match = url.match(regExp);
            return (match && match[2].length === 11) ? match[2] : null;
        }

        function toggleAdminMode() {
            const pass = prompt("Password admin:");
            if (pass === ADMIN_PASS) {
                isAdmin = true;
                document.getElementById('loginAdminBtn').innerText = "Logout Admin";
                showToast("Mode Admin Aktif");
            } else if(pass) {
                alert("Password salah!");
            }
        }

        function addPost() {
            const quote = document.getElementById('quoteInput').value;
            const song = document.getElementById('songInput').value;

            if(quote.trim() === "" && song.trim() === "") {
                alert("Mohon isi minimal kutipan atau lagu!");
                return;
            }

            window.firebaseFunctions.push(postsRef, {
                quote: quote,
                song: song,
                createdAt: Date.now()
            });

            document.getElementById('quoteInput').value = "";
            document.getElementById('songInput').value = "";
            showToast("Berhasil diposting!");
        }

        function deletePost(key) {
            if(!isAdmin) return;
            if(confirm("Yakin hapus?")) {
                window.firebaseFunctions.remove(window.firebaseFunctions.ref(db, 'posts/' + key));
                showToast("Dihapus!");
            }
        }

        function renderPosts(dataObj) {
            const container = document.getElementById('postsContainer');
            container.innerHTML = "";
            
            if(!dataObj) {
                container.innerHTML = "<p style='grid-column:1/-1;text-align:center;color:#8D6E63'>Belum ada postingan. Jadilah yang pertama!</p>";
                return;
            }

            // Urutkan terbaru dulu
            const entries = Object.entries(dataObj).reverse();

            entries.forEach(([key, post]) => {
                const card = document.createElement('div');
                card.className = 'card';
                
                // Cek apakah link YouTube
                const youtubeID = getYouTubeID(post.song);
                let musicContent = '';
                
                if(youtubeID) {
                    // Tampilkan Player YouTube
                    musicContent = `
                        <div class="music-player">
                            <iframe src="https://www.youtube.com/embed/${youtubeID}?theme=dark" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
                        </div>
                    `;
                } else if(post.song && post.song.trim() !== "") {
                    // Tampilkan teks biasa
                    musicContent = `<div class="song-text">🎵 ${post.song}</div>`;
                } else {
                    musicContent = '';
                }

                card.innerHTML = `
                    <div class="quote-text">"${post.quote}"</div>
                    ${musicContent}
                    <button class="btn-delete" style="display:${isAdmin?'block':'none'}" onclick="deletePost('${key}')">Hapus Postingan</button>
                `;
                container.appendChild(card);
            });
        }

        function showToast(msg) {
            const x = document.getElementById("toast");
            x.innerText = msg;
            x.className = "show";
            setTimeout(() => x.className = x.className.replace("show", ""), 3000);
        }
    </script>
</body>
</html>
