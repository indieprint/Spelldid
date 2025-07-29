<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Grammar Checker</title>
  <link rel="stylesheet" href="style.css" />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap" rel="stylesheet">
</head>
<body>

  <div class="background"></div>

  <div class="container">
    <h1>✨ Grammar Checker with Auto-Correct</h1>
    <textarea id="textInput" placeholder="Write something with grammar or tense issues..."></textarea>
    
    <div class="buttons">
      <button onclick="checkGrammar()">Check Grammar</button>
      <button onclick="autoCorrect()">Auto-Correct</button>
    </div>

    <div id="output" class="suggestions"></div>
  </div>

  <script>
    async function checkGrammar(autoCorrectMode = false) {
      const inputArea = document.getElementById("textInput");
      const text = inputArea.value;
      const outputDiv = document.getElementById("output");
      outputDiv.innerHTML = "<div class='typing'>🧠 Checking grammar...</div>";

      const response = await fetch("https://api.languagetool.org/v2/check", {
        method: "POST",
        headers: { "Content-Type": "application/x-www-form-urlencoded" },
        body: new URLSearchParams({
          text: text,
          language: "en-US"
        })
      });

      const result = await response.json();
      const matches = result.matches;

      if (matches.length === 0) {
        outputDiv.innerHTML = "<strong>✅ No grammar issues found!</strong>";
        return;
      }

      if (autoCorrectMode) {
        let correctedText = text;
        matches.sort((a, b) => b.offset - a.offset);
        matches.forEach(match => {
          if (match.replacements.length > 0) {
            const suggestion = match.replacements[0].value;
            correctedText =
              correctedText.slice(0, match.offset) +
              suggestion +
              correctedText.slice(match.offset + match.length);
          }
        });
        inputArea.value = correctedText;
        outputDiv.innerHTML = "<strong>✅ Text auto-corrected.</strong>";
        return;
      }

      outputDiv.innerHTML = "<strong>📝 Grammar Suggestions:</strong><br><br>";
      matches.forEach(match => {
        const issue = document.createElement("div");
        issue.classList.add("issue");
        issue.innerHTML = `
          <div><strong>Issue:</strong> ${match.message}</div>
          <div><strong>Suggestion:</strong> ${match.replacements.map(r => r.value).join(", ") || "No suggestion"}</div>
          <div><strong>Context:</strong> ${highlightMatch(match.context.text, match.context.offset, match.context.length)}</div>
        `;
        outputDiv.appendChild(issue);
      });
    }

    function highlightMatch(text, offset, length) {
      const before = text.slice(0, offset);
      const match = text.slice(offset, offset + length);
      const after = text.slice(offset + length);
      return `${before}<span class='highlight'>${match}</span>${after}`;
    }

    function autoCorrect() {
      checkGrammar(true);
    }
  </script>

</body>
</html>
