<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>محرر النصوص المطور</title>
    <style>
        :root {
            --primary-color: #007bff;
            --dark-green-bg: #004d40;
            --dark-text: #333;
            --white: #ffffff;
            --danger-color: #dc3545;
            --border-color: #333;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--dark-green-bg);
            margin: 0;
            padding: 20px;
            color: var(--dark-text);
            transition: background-color 0.3s;
        }

        .container { max-width: 900px; margin: 0 auto; }
        .hidden { display: none !important; }

        #addPostBtn {
            position: fixed;
            bottom: 25px;
            left: 25px;
            background-color: var(--primary-color);
            color: white;
            border: none;
            border-radius: 50%;
            width: 60px;
            height: 60px;
            font-size: 36px;
            cursor: pointer;
            display: flex;
            justify-content: center;
            align-items: center;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
            z-index: 1000;
            transition: transform 0.2s;
        }
        #addPostBtn:hover { transform: scale(1.1); }

        #editorPage {
            background-color: var(--white);
            border-radius: 8px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
            padding: 25px;
            margin-bottom: 100px;
        }

        #postTitle {
            width: 100%;
            padding: 12px;
            font-size: 24px;
            font-weight: bold;
            border: none;
            border-bottom: 2px solid #eee;
            margin-bottom: 20px;
            box-sizing: border-box;
            outline: none;
        }
        #postTitle:focus { border-bottom-color: var(--primary-color); }

        .toolbar {
            background-color: #f8f9fa;
            border: 1px solid #ddd;
            border-radius: 5px;
            padding: 8px;
            margin-bottom: 15px;
            display: flex;
            flex-wrap: nowrap;
            overflow-x: auto;
            white-space: nowrap;
        }
        .toolbar button, .toolbar select {
            background-color: transparent;
            border: 1px solid transparent;
            padding: 8px 10px;
            cursor: pointer;
            border-radius: 3px;
            font-size: 16px;
            flex-shrink: 0;
        }
        .toolbar button:hover, .toolbar select:hover { background-color: #e2e6ea; }
        
        #postContent {
            width: 100%;
            min-height: 400px;
            border: 1px solid #ddd;
            padding: 15px;
            font-size: 16px;
            line-height: 1.6;
            box-sizing: border-box;
            border-radius: 5px;
            outline: none;
            overflow-y: auto;
        }
        #postContent:focus { border-color: var(--primary-color); }
        #postContent img { max-width: 100%; height: auto; cursor: pointer; border-radius: 4px; box-shadow: 0 0 5px rgba(0,0,0,0.2); }
        
        .editor-actions { margin-top: 20px; text-align: left; }
        .editor-actions button { padding: 10px 20px; border: none; border-radius: 5px; font-size: 16px; cursor: pointer; margin-right: 10px; }
        #savePostBtn { background-color: #28a745; color: white; }
        #cancelEditBtn { background-color: #6c757d; color: white; }

        #postsContainer {
            display: flex;
            flex-direction: column;
            gap: 20px;
            padding-bottom: 100px;
        }

        .post-card {
            background-color: var(--white);
            border: 2px solid var(--border-color);
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            padding: 20px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
        }
        
        .post-card .post-title {
            margin: 0;
            font-size: 20px;
            font-weight: 700;
            color: #0056b3;
            line-height: 1.4;
            word-break: break-word; 
            margin-bottom: 15px; /* إضافة مسافة بين العنوان والأزرار */
        }
        
        .post-actions { display: flex; gap: 8px; flex-wrap: wrap; justify-content: flex-start; }
        .post-actions button { padding: 6px 12px; border: 1px solid #ddd; background-color: transparent; border-radius: 5px; cursor: pointer; font-size: 14px; }

        @media (max-width: 768px) {
            body { padding: 0; }
            .container { width: 100%; padding: 0; }
            #editorPage { border-radius: 0; padding: 15px; min-height: 100vh; box-sizing: border-box; }
            #postsContainer { padding: 15px; padding-bottom: 100px; }
            .post-card { min-height: 20vh; max-height: 25vh; }
        }
    </style>
</head>
<body>
    
    <button id="addPostBtn" title="إضافة مقال جديد">+</button>

    <div class="container">
        <div id="editorPage" class="hidden">
            <input type="text" id="postTitle" placeholder="عنوان المقال هنا...">
            <div class="toolbar">
                <select onchange="formatDoc('formatBlock', this.value)">
                    <option value="p">نص عادي</option><option value="h1">عنوان 1</option><option value="h2">عنوان 2</option><option value="h3">عنوان 3</option>
                </select>
                <button onclick="formatDoc('bold')"><b>B</b></button><button onclick="formatDoc('italic')"><i>I</i></button><button onclick="formatDoc('underline')"><u>U</u></button><button onclick="formatDoc('strikeThrough')"><del>S</del></button><button onclick="formatDoc('insertUnorderedList')">●</button><button onclick="formatDoc('insertOrderedList')">1.</button><button onclick="addLink()">🔗</button><button onclick="triggerImageUpload()">🖼️</button>
                <input type="file" id="imageUpload" class="hidden" accept="image/*" onchange="insertImage(event)">
            </div>
            <div id="postContent" contenteditable="true" dir="auto"></div>
            <div class="editor-actions"><button id="savePostBtn">حفظ المقال</button><button id="cancelEditBtn">إلغاء</button></div>
        </div>
        <div id="postsContainer"></div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const editorPage = document.getElementById('editorPage');
            const postsContainer = document.getElementById('postsContainer');
            const addPostBtn = document.getElementById('addPostBtn');
            const savePostBtn = document.getElementById('savePostBtn');
            const cancelEditBtn = document.getElementById('cancelEditBtn');
            const postTitleInput = document.getElementById('postTitle');
            const postContentDiv = document.getElementById('postContent');
            const imageUploadInput = document.getElementById('imageUpload');
            
            let posts = [];
            let currentEditingId = null;

            function renderPosts() {
                postsContainer.innerHTML = '';
                posts.forEach(post => {
                    const postCard = document.createElement('div');
                    postCard.className = 'post-card';
                    postCard.setAttribute('data-id', post.id);

                    // --- تم تعديل هذا الجزء ---
                    postCard.innerHTML = `
                        <h2 class="post-title">${post.title}</h2>
                        <div class="post-actions">
                            <button onclick="handleEdit('${post.id}')" title="تعديل">✏️ تعديل</button>
                            <button onclick="handleDelete('${post.id}')" title="حذف">🗑️ حذف</button>
                            <button onclick="handleDownloadDoc('${post.id}')" title="تحميل DOC">📄 Doc</button>
                        </div>
                    `;
                    // --- نهاية التعديل ---
                    postsContainer.appendChild(postCard);
                });
            }

            function showEditor() {
                postsContainer.classList.add('hidden');
                editorPage.classList.remove('hidden');
                document.body.style.backgroundColor = 'var(--white)';
            }

            function showHomePage() {
                editorPage.classList.add('hidden');
                postsContainer.classList.remove('hidden');
                document.body.style.backgroundColor = 'var(--dark-green-bg)';
            }
            
            function clearEditor() {
                postTitleInput.value = '';
                postContentDiv.innerHTML = '';
                currentEditingId = null;
            }

            function loadPostsFromStorage() {
                const storedPosts = localStorage.getItem('mySimpleBlogPosts');
                posts = storedPosts ? JSON.parse(storedPosts) : [];
                renderPosts();
            }

            function savePostsToStorage() {
                localStorage.setItem('mySimpleBlogPosts', JSON.stringify(posts));
            }

            addPostBtn.addEventListener('click', () => { clearEditor(); showEditor(); });
            cancelEditBtn.addEventListener('click', () => { clearEditor(); showHomePage(); });

            savePostBtn.addEventListener('click', () => {
                const title = postTitleInput.value.trim();
                const content = postContentDiv.innerHTML.trim();
                if (!title || !content) { alert('يرجى كتابة العنوان والمحتوى قبل الحفظ.'); return; }
                if (currentEditingId) {
                    const postIndex = posts.findIndex(p => p.id === currentEditingId);
                    if (postIndex > -1) { posts[postIndex] = { ...posts[postIndex], title, content }; }
                } else {
                    posts.unshift({ id: Date.now().toString(), title, content });
                }
                savePostsToStorage(); renderPosts(); clearEditor(); showHomePage();
            });
            
            postContentDiv.addEventListener('click', function(event) {
                if (event.target.tagName === 'IMG') {
                    const img = event.target;
                    const newWidth = prompt('أدخل عرض الصورة الجديد (مثال: 50% أو 300px):', img.style.width || '80%');
                    if (newWidth) { img.style.width = newWidth; img.style.height = 'auto'; }
                }
            });

            window.handleEdit = (id) => {
                const postToEdit = posts.find(p => p.id === id);
                if (postToEdit) { currentEditingId = id; postTitleInput.value = postToEdit.title; postContentDiv.innerHTML = postToEdit.content; showEditor(); }
            }

            window.handleDelete = (id) => {
                if (confirm('هل أنت متأكد من حذف هذا المقال؟')) { posts = posts.filter(p => p.id !== id); savePostsToStorage(); renderPosts(); }
            }

            window.handleDownloadDoc = (id) => {
                const post = posts.find(p => p.id === id);
                if (!post) return;
                const sourceHTML = `<html><head><meta charset='utf-8'></head><body><h1>${post.title}</h1>${post.content}</body></html>`;
                const source = 'data:application/vnd.ms-word;charset=utf-8,' + encodeURIComponent(sourceHTML);
                const a = document.createElement("a");
                a.href = source;
                a.download = `${post.title.replace(/\s/g, '_')}.doc`;
                a.click();
            }

            window.formatDoc = (command, value = null) => { document.execCommand(command, false, value); postContentDiv.focus(); }
            window.addLink = () => { const url = prompt('أدخل رابط URL:'); if (url) formatDoc('createLink', url); }
            window.triggerImageUpload = () => imageUploadInput.click();
            window.insertImage = (event) => {
                const file = event.target.files[0];
                if (file) {
                    const reader = new FileReader();
                    reader.onload = (e) => formatDoc('insertHTML', `<img src="${e.target.result}" style="width: 80%; height: auto;">`);
                    reader.readAsDataURL(file);
                }
                event.target.value = null;
            }

            loadPostsFromStorage();
            showHomePage();
        });
    </script>
</body>
</html>
