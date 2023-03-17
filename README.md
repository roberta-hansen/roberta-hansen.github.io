<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Ask a Question</title>
  <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <style>
    body {
      font-family: Arial, sans-serif;
    }

    label {
      display: block;
      font-weight: bold;
      margin-bottom: 10px;
    }

    textarea {
      width: 100%;
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 5px;
      resize: none;
    }

    button {
      padding: 10px;
      background-color: #007bff;
      color: #fff;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      margin-right: 10px;
    }

    button:hover {
      background-color: #0062cc;
    }

    #answer {
      margin-top: 10px;
    }
  </style>
</head>
<body>
 <p>This model has a maximum context length of 4097 tokens.</p>
 <p>Both the question and answer are included in the 4097 tokens.</p>
  <div>
    <label for="question">What is your question?</label>
    <textarea id="question" rows="4" cols="50"></textarea>
  </div>
  <br>
  <div>
    <button id="answer-btn">Answer</button>
    <button id="clear-btn">Clear</button>
  </div>
  <br>
  <div>
    <label for="answer">Answer:</label>
    <br>
    <textarea id="answer" rows="10" cols="50" readonly></textarea>
  </div>
  </div>
  <div id="error-message"></div>
  <script>
    $(document).ready(function() {
    
      var chatgpt_api;
      var apiUrl = "https://api.openai.com/v1/completions";
    
      function askApiKey() {
        chatgpt_api = prompt("Please enter your OpenAI API key:");
        if (!chatgpt_api) {
          alert("API key cannot be empty. Please try again.");
          askApiKey();
        } else {
          document.cookie = "chatgpt_api=" + chatgpt_api + "; path=/";
        }
      }
    
      if (document.cookie.indexOf("chatgpt_api") >= 0) {
        chatgpt_api = document.cookie.replace(/(?:(?:^|.*;\s*)chatgpt_api\s*\=\s*([^;]*).*$)|^.*$/, "$1");
      } else {
        askApiKey();
      }
    
      $('#answer-btn').click(function() {
        var question = $('#question').val();
        var body = {
          prompt: question,
          model: "text-davinci-003",
          max_tokens: 4000,
          temperature: 1.0
        };
        var headers = {
          "Content-Type": "application/json",
          "Authorization": "Bearer " + chatgpt_api
        };
        var jsonBody = JSON.stringify(body);
        $.ajax({
          url: apiUrl,
          type: "POST",
          headers: headers,
          data: jsonBody,
          success: function(response) {
            var answer = response.choices.map(function(choice) { return choice.text; }).join("\n");
            $('#answer').val(answer);
          },
          error: function(jqXHR, textStatus, errorThrown) {
            var errorMsg = "Error: ";
            if (jqXHR.status === 400) {
              errorMsg += "You  have exceeded the limit of 4097 tokens.";
            } else if (jqXHR.status === 401) {
              errorMsg += "Unauthorized. Invalid API key.";
            } else {
              errorMsg += textStatus + " " + errorThrown;
            }
            console.error(errorMsg);
            $('#answer').css('color', 'red').val(errorMsg);
          }
        });
      });
    
      $('#clear-btn').click(function() {
        $('#question').val('');
        $('#answer').val('');
      });
    
    });
      </script>
</body>
</html>
