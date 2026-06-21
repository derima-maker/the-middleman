<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Firebase Data Provider</title>
  <style>
    body {
      font-family: sans-serif;
      background: #1a1a1a;
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
    }
    .loader {
      text-align: center;
    }
    .loader span {
      display: inline-block;
      width: 10px;
      height: 10px;
      background: #FFC107;
      border-radius: 50%;
      margin: 0 4px;
      animation: bounce 0.8s infinite alternate;
    }
    .loader span:nth-child(2) { animation-delay: 0.2s; }
    .loader span:nth-child(3) { animation-delay: 0.4s; }
    @keyframes bounce {
      to { transform: translateY(-10px); }
    }
  </style>
</head>
<body>
  <div class="loader">
    <p>Loading data from Firebase...</p>
    <div>
      <span></span><span></span><span></span>
    </div>
  </div>

  <!-- Firebase SDK -->
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>

  <script>
    // 🔥 YOUR FIREBASE CONFIGURATION
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT.firebaseapp.com",
      projectId: "YOUR_PROJECT_ID",
      storageBucket: "YOUR_PROJECT.appspot.com",
      messagingSenderId: "YOUR_SENDER_ID",
      appId: "YOUR_APP_ID"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();

    async function fetchData() {
      try {
        const videos = [];
        const audios = [];

        // Example: read from a collection named "content"
        const snapshot = await db.collection("content")
          .orderBy("created_at", "desc")
          .get();

        snapshot.forEach(doc => {
          const data = doc.data();
          const item = {
            id: doc.id,
            title: data.title || "Untitled",
            artist: data.artist || "Unknown",
            pastor: data.artist || "Unknown",
            url: data.url,
            type: data.type || "video",
            views: data.views || 0,
            category: data.category || "sermon",
            duration: data.duration || "0:00",
            is_status: data.is_status || false,
            expires_at: data.expires_at || null,
          };
          if (data.type === "video") videos.push(item);
          else if (data.type === "audio") audios.push(item);
        });

        // Send data to parent window (the main app)
        const message = {
          type: "FIREBASE_DATA",
          payload: { videos, audios }
        };
        window.parent.postMessage(message, "*");

        // Also store in localStorage as fallback
        localStorage.setItem("firebaseData", JSON.stringify({ videos, audios }));
        console.log("✅ Firebase data sent to parent");
      } catch (err) {
        console.error("❌ Firebase fetch error:", err);
        // Send error to parent
        window.parent.postMessage({ type: "FIREBASE_ERROR", error: err.message }, "*");
      }
    }

    // Wait for Firestore to be ready
    fetchData();
  </script>
</body>
</html>
