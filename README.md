# entry-search
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <title>엔트리 유저 검색기</title>
  <style>
    body { font-family: sans-serif; padding: 20px; }
    input, button { font-size: 16px; padding: 8px; }
    .user-link {
      display: block;
      margin: 10px 0;
      color: blue;
      text-decoration: underline;
    }
  </style>
</head>
<body>
  <h2>🔍 엔트리 유저 검색기</h2>
  <input type="text" id="nickname" placeholder="닉네임 입력" />
  <button id="searchBtn">검색</button>
  <div id="result" style="margin-top: 20px;"></div>

  <script>
    // 연속 일치 문자 개수로 유사도 측정
    function similarity(a, b) {
      a = a.toLowerCase();
      b = b.toLowerCase();
      let matches = 0;
      for (let i = 0; i < Math.min(a.length, b.length); i++) {
        if (a[i] === b[i]) matches++;
        else break; // 첫 불일치 시 종료
      }
      return matches;
    }

    const btn = document.getElementById('searchBtn');
    const input = document.getElementById('nickname');
    const resultBox = document.getElementById('result');

    async function searchUser() {
      const nickname = input.value.trim();
      if (!nickname) {
        resultBox.textContent = '❗ 닉네임을 입력해주세요.';
        return;
      }

      btn.disabled = true;
      resultBox.textContent = '🔎 검색 중...';

      // 테스트용 CORS 프록시 (실환경에서는 안정적 프록시 권장)
      const proxy = "https://corsproxy.io/?";
      const url = `https://playentry.org/api/user/find?q=${encodeURIComponent(nickname)}`;

      try {
        const res = await fetch(proxy + url);
        if (!res.ok) throw new Error(`HTTP 오류: ${res.status}`);

        const users = await res.json();

        if (!Array.isArray(users) || users.length === 0) {
          resultBox.textContent = "😢 해당 닉네임을 가진 사용자가 없습니다.";
          btn.disabled = false;
          return;
        }

        users.sort((a, b) => similarity(b.nickname, nickname) - similarity(a.nickname, nickname));

        resultBox.innerHTML = "<h3>🔗 유사 닉네임 검색 결과:</h3>";
        users.forEach(user => {
          const link = document.createElement("a");
          link.href = `https://playentry.org/profile/${user._id}`;
          link.textContent = `🧑 ${user.nickname}`;
          link.className = "user-link";
          link.target = "_blank";
          resultBox.appendChild(link);
        });
      } catch (err) {
        console.error(err);
        resultBox.textContent = "⚠️ 오류 발생 (인터넷 문제 또는 Entry 서버 문제일 수 있음)";
      } finally {
        btn.disabled = false;
      }
    }

    btn.addEventListener('click', searchUser);
    input.addEventListener('keydown', e => {
      if (e.key === 'Enter') searchUser();
    });
  </script>
</body>
</html>
