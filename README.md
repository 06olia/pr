<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Конкурентні запити з таймаутом</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
    }
    pre {
      background : #f4f4f4;
      padding: 10px;
      border-radius: 5px;
      overflow-x: auto;
    }
  </style>
</head>
<body>
  <h1>Результати конкурентних запитів</h1>
  <pre id="output">Виконується...</pre>
  <script>
    function fetchWithTimeout(url, timeout) {
      const fetchPromise = fetch(url); 
      const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => reject(new Error("Request timeout")), timeout); 
      });
      return Promise.race([fetchPromise, timeoutPromise]);
    }
    async function fetchData() {
      const urls = [
        "https://jsonplaceholder.typicode.com/posts/1",
        "https://jsonplaceholder.typicode.com/users/1",
      ];
      const timeout = 2000; 
      try {
        const results = await Promise.allSettled(
          urls.map((url) => fetchWithTimeout(url, timeout))
        );
        let output = "";
        results.forEach((result, index) => {
          if (result.status === "fulfilled") {
            output += `Запит до ${urls[index]} успішний:\n${JSON.stringify(result.value, null, 2)}\n\n`;
          } else {
            output += `Запит до ${urls[index]} відхилено: ${result.reason.message}\n\n`;
          }
        });
        if (results.some((result) => result.reason && result.reason.message === "Request timeout")) {
          output += "Один із запитів завершено через таймаут.\n";
        } else {
          output += "Усі запити завершено успішно.\n";
        }
        document.getElementById("output").textContent = output;
      } catch (error) {
        console.error("Помилка:", error);
        document.getElementById("output").textContent = "Помилка при виконанні запитів.";
      }
    }
   </script>
</head>
<body>  
</body>
</html>
