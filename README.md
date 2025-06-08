<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QuizCraft - Client-Side Quiz Generator</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            color: black;
            max-width: 1000px;
            margin: 0 auto;
            padding: 20px;
            background-color: white;
        }
        
        h1 {
            color: black;
            text-align: center;
            margin-bottom: 10px;
        }
        .loader {
           border: 4px solid black;
           border-top: 4px solid lightskyblue;
           border-radius: 50%;
           width: 30px;
           height: 30px;
           animation: spin 1s linear infinite;
           margin: 20px auto;
           display: none; 
}
         @keyframes spin {
          0% { transform: rotate(0deg); }
          100% { transform: rotate(360deg); }
         }
        .subtitle {
            text-align: center;
            color: black;
            margin-bottom: 30px;
        }
        
        .input-section {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            margin-bottom: 20px;
        }
        
        #text-input {
            width: 100%;
            min-height: 150px;
            padding: 10px;
            border: 1px solid black;
            border-radius: 4px;
            font-family: inherit;
            margin-bottom: 15px;
            resize: vertical;
        }
        
        .actions {
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
        }
        
        button {
            background-color: lightskyblue;
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            transition: background-color 0.3s;
        }
        
        button:hover {
            background-color: lightblue;
        }
        
        button:disabled {
            background-color: bisque;
            cursor: auto;
        }
        
        .output-tabs {
            display: flex;
            gap: 5px;
            margin-bottom: 15px;
            flex-wrap: wrap;
        }
        
        .tab-btn {
            background-color: white;
            color: grey;
            text-decoration : black;
        }
        
        .tab-btn.active {
            background-color: lightskyblue;
            color: white;
        }
        
        .tab-content {
            display: none;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        
        .tab-content.active {
            display: block;
        }
        
        .mcq-item, .blank-item {
            margin-bottom: 20px;
            padding-bottom: 15px;
            border-bottom: 1px solid white;
        }
        
        .question {
            font-weight: bold;
            margin-bottom: 10px;
        }
        
        .answer {
            color: darkgray;
            font-weight: bold;
            margin-top: 5px;
        }
        
        .blank {
            background-color: white;
            padding: 2px 5px;
            border-radius: 3px;
            border: 1px ;
        }
        
        #keypoints-output li {
            margin-bottom: 8px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>QuizCraft</h1>
        <p class="subtitle">Transform any text into interactive learning materials</p>
        
        <div id="loading" class="loader" style="display:none;"></div>
        <div class="input-section">
            <textarea id="text-input" placeholder="Paste your article, textbook content, or any educational material here..."></textarea>
            <div class="actions">
                <button id="generate-btn">Generate Materials</button>
                <button id="clear-btn">Clear</button>
                <input type="file" id="file-upload" accept=".txt,.pdf">
                <button id="export-pdf">Export PDF</button>
            </div>
        </div>
        
        <div class="output-tabs">
            <button class="tab-btn active" data-tab="mcq">MCQs</button>
            <button class="tab-btn" data-tab="blanks">Fill-in-Blanks</button>
            <button class="tab-btn" data-tab="summary">Summary</button>
            <button class="tab-btn" data-tab="keypoints">Key Points</button>
        </div>
        
        <div class="output-content">
            <div id="mcq-output" class="tab-content active"></div>
            <div id="blanks-output" class="tab-content"></div>
            <div id="summary-output" class="tab-content"></div>
            <div id="keypoints-output" class="tab-content"></div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/jspdf@latest/dist/jspdf.umd.min.js"></script>
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            
            const textInput = document.getElementById('text-input');
            const generateBtn = document.getElementById('generate-btn');
            const clearBtn = document.getElementById('clear-btn');
            const fileUpload = document.getElementById('file-upload');
            const exportBtn = document.getElementById('export-pdf');
            
            document.querySelectorAll('.tab-btn').forEach(btn => {
                btn.addEventListener('click', () => {
                    document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
                    document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
                    
                    btn.classList.add('active');
                    document.getElementById(`${btn.dataset.tab}-output`).classList.add('active');
                });
            });
            
            generateBtn.addEventListener('click', () => {
                const text = textInput.value.trim();
                if (!text) return alert('Please enter some text first');
                
                generateBtn.disabled = true;
                generateBtn.textContent = 'Processing...';
                
                setTimeout(() => {
                    const results = processText(text);
                    displayResults(results);
                    generateBtn.disabled = false;
                    generateBtn.textContent = 'Generate Materials';
                }, 1000);
            });
            
           fileUpload.addEventListener('change', (e) => {
                const file = e.target.files[0];
                if (!file) return;
                
                const reader = new FileReader();
                reader.onload = (event) => {
                    textInput.value = event.target.result;
                };
                reader.readAsText(file);
            });
            
           clearBtn.addEventListener('click', () => {
                textInput.value = '';
                document.querySelectorAll('.tab-content').forEach(c => c.innerHTML = '');
            });
            
            exportBtn.addEventListener('click', exportToPDF);
                   
            function processText(text) {
                 const mcqs = [
                    {
                        question: "The ______ protocol is used to secure HTTP connections.",
                        options: ["HTTPS", "FTP", "SMTP", "TCP"],
                        answer: "HTTPS"
                    },
                    {
                        question: "Python uses ______ to define code blocks instead of curly braces.",
                        options: ["indentation", "parentheses", "square brackets", "semicolons"],
                        answer: "indentation"
                    },
                    {
                        question: "The ______ layer in the OSI model handles routing.",
                        options: ["Network", "Transport", "Data Link", "Session"],
                        answer: "Network"
                    },
                    {
                        question: "______ is the process of converting code into machine language.",
                        options: ["Compilation", "Interpretation", "Transpilation", "Debugging"],
                        answer: "Compilation"
                    },
                    {
                        question: "In REST APIs, ______ is used to retrieve resource data.",
                        options: ["GET", "POST", "PUT", "DELETE"],
                        answer: "GET"
                    },
                    {
                        question: "______ is a JavaScript runtime built on Chrome's V8 engine.",
                        options: ["Node.js", "Django", "Laravel", "Flask"],
                        answer: "Node.js"
                    }
                ];
        
                const blanks = [
                    {
                        text: "The _____ keyword in Python creates a new class.",
                        answer: "class"
                    },
                    {
                        text: "HTTP status code 404 means _____.",
                        answer: "Not Found"
                    },
                    {
                        text: "_____ is the process of hiding implementation details in OOP.",
                        answer: "Encapsulation"
                    },
                    {
                        text: "The _____ data structure follows LIFO principle.",
                        answer: "stack"
                    },
                    {
                        text: "_____ is a version control system created by Linus Torvalds.",
                        answer: "Git"
                    },
                    {
                        text: "The _____ loop in Python iterates over a sequence.",
                        answer: "for"
                    }
                ];
        
               const sentences = text.match(/[^.!?]+[.!?]+/g) || [text];
                const summary = sentences.slice(0, 3).join(' ') + (sentences.length > 3 ? '...' : '');
                
                const keypoints = sentences
                    .filter(s => s.length > 30 && /[A-Z]/.test(s[0]))
                    .slice(0, 5)
                    .map(s => s.trim().replace(/[.!?]+$/, ''));
        
                return { mcqs, blanks, summary, keypoints };
            }
        
            function displayResults(results) {
                document.getElementById('mcq-output').innerHTML = 
                    results.mcqs.map((q, i) => `
                        <div class="mcq-item">
                            <p class="question">${i+1}. ${q.question}</p>
                            <ol type="A">
                                ${q.options.map(opt => `<li>${opt}</li>`).join('')}
                            </ol>
                            <p class="answer">Answer: ${q.answer}</p>
                        </div>
                    `).join('');
        
                document.getElementById('blanks-output').innerHTML = 
                    results.blanks.map((b, i) => `
                        <div class="blank-item">
                            <p>${i+1}. ${b.text.replace('_____', '<span class="blank">_____</span>')}</p>
                            <p class="answer">Answer: ${b.answer}</p>
                        </div>
                    `).join('');
        
                document.getElementById('summary-output').textContent = results.summary;
                document.getElementById('keypoints-output').innerHTML = 
                    results.keypoints.map(k => `<li>${k}</li>`).join('');
            }
        
            function exportToPDF() {
                const activeTab = document.querySelector('.tab-content.active');
                if (!activeTab || !activeTab.textContent.trim()) {
                    return alert('Generate content first before exporting');
                }
                
                const { jsPDF } = window.jspdf;
                const doc = new jsPDF();
                
                doc.text(activeTab.textContent, 10, 10, { maxWidth: 180 });
                doc.save('quizcraft-export.pdf');
            }
        });
        </script>
</body>
</html>
