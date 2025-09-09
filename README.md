<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <title>To-Do List dengan Group/Category, Timestamp, No SJ, dan Tanggal Barang Sampai</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 950px;
      margin: 50px auto;
      padding: 20px;
      background: #f5f5f5;
      border-radius: 8px;
    }
    h1 {
      text-align: center;
      color: #333;
      margin-bottom: 30px;
    }
    .input-group {
      margin-bottom: 20px;
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
    }
    input[type="text"], select, input[type="date"] {
      padding: 10px;
      font-size: 16px;
      box-sizing: border-box;
      border-radius: 4px;
      border: 1px solid #ccc;
      flex-grow: 1;
    }
    button {
      padding: 10px 14px;
      font-size: 16px;
      cursor: pointer;
      border: none;
      background: #007bff;
      color: white;
      border-radius: 4px;
      flex-shrink: 0;
    }
    button:hover {
      background: #0056b3;
    }
    #categoriesContainer {
      display: flex;
      gap: 20px;
      flex-wrap: wrap;
      justify-content: center;
    }
    .category-section {
      background: #eaeaea;
      border-radius: 8px;
      padding: 15px;
      width: 320px;
      box-sizing: border-box;
      display: flex;
      flex-direction: column;
    }
    .category-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 10px;
    }
    .category-header input {
      width: 70%;
      padding: 6px;
      font-size: 16px;
      border-radius: 4px;
      border: 1px solid #ccc;
    }
    .category-header button {
      background: #dc3545;
      padding: 6px 10px;
      font-size: 14px;
      border-radius: 4px;
      border: none;
      cursor: pointer;
      margin-left: 8px;
      white-space: nowrap;
    }
    .category-header button:hover {
      background: #b52a37;
    }
    ul {
      list-style: none;
      padding-left: 0;
      margin: 0;
      flex-grow: 1;
      overflow-y: auto;
      max-height: 350px;
    }
    li {
      background: white;
      padding: 8px 10px;
      margin-bottom: 6px;
      border-radius: 4px;
      display: flex;
      flex-direction: column;
      font-size: 16px;
    }
    li .todo-top {
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    li .todo-text {
      flex-grow: 1;
      margin-left: 8px;
      user-select: none;
    }
    li.completed .todo-text {
      text-decoration: line-through;
      color: #777;
    }
    li button {
      background: #007bff;
      padding: 6px 10px;
      font-size: 14px;
      border-radius: 4px;
      border: none;
      cursor: pointer;
      white-space: nowrap;
      margin-left: 10px;
    }
    li button:hover {
      background: #0056b3;
    }
    .timestamps {
      font-size: 12px;
      color: #555;
      margin-top: 4px;
      font-style: italic;
    }
  </style>
</head>
<body>

  <h1>To-Do List dengan Group/Category dan Timestamp Otomatis</h1>

  <div class="input-group">
    <input type="text" id="categoryInput" placeholder="Tambah / edit kategori baru..." />
    <button id="addCategoryBtn">Tambah Kategori</button>
  </div>

  <div class="input-group">
    <input type="text" id="todoInput" placeholder="Tambah tugas baru..." />
    <input type="text" id="noSJInput" placeholder="No SJ" />
    <input type="date" id="tglBarangSampaiInput" />
    <select id="categorySelect"></select>
    <button id="addTodoBtn">Tambah Tugas</button>
  </div>

  <div id="categoriesContainer"></div>

  <script>
    const categoryInput = document.getElementById('categoryInput');
    const addCategoryBtn = document.getElementById('addCategoryBtn');
    const todoInput = document.getElementById('todoInput');
    const noSJInput = document.getElementById('noSJInput');
    const tglBarangSampaiInput = document.getElementById('tglBarangSampaiInput');
    const categorySelect = document.getElementById('categorySelect');
    const addTodoBtn = document.getElementById('addTodoBtn');
    const categoriesContainer = document.getElementById('categoriesContainer');

    let categories = JSON.parse(localStorage.getItem('categories')) || [];

    function saveCategories() {
      localStorage.setItem('categories', JSON.stringify(categories));
    }

    function renderCategorySelect() {
      categorySelect.innerHTML = '';
      categories.forEach(cat => {
        const option = document.createElement('option');
        option.value = cat.id;
        option.textContent = cat.name;
        categorySelect.appendChild(option);
      });
    }

    function formatDateTime(dateStr) {
      if(!dateStr) return '';
      const d = new Date(dateStr);
      return d.toLocaleString('id-ID', { 
        year: 'numeric', month: '2-digit', day: '2-digit',
        hour: '2-digit', minute: '2-digit'
      });
    }

    function formatDateOnly(dateStr) {
      if(!dateStr) return '';
      const d = new Date(dateStr);
      return d.toLocaleDateString('id-ID', { year: 'numeric', month: '2-digit', day: '2-digit' });
    }

    function renderCategories() {
      categoriesContainer.innerHTML = '';
      categories.forEach(cat => {
        const section = document.createElement('div');
        section.className = 'category-section';

        const header = document.createElement('div');
        header.className = 'category-header';

        const input = document.createElement('input');
        input.type = 'text';
        input.value = cat.name;
        input.addEventListener('change', e => {
          const newName = e.target.value.trim();
          if(newName) {
            cat.name = newName;
            saveCategories();
            renderCategorySelect();
            renderCategories();
          } else {
            e.target.value = cat.name;
          }
        });

        const delBtn = document.createElement('button');
        delBtn.textContent = 'Hapus Kategori';
        delBtn.onclick = () => {
          if(confirm(`Hapus kategori "${cat.name}"? Semua tugas di dalamnya juga akan terhapus.`)) {
            categories = categories.filter(c => c.id !== cat.id);
            saveCategories();
            renderCategorySelect();
            renderCategories();
          }
        };

        header.appendChild(input);
        header.appendChild(delBtn);
        section.appendChild(header);

        const ul = document.createElement('ul');

        cat.todos.forEach((todo, idx) => {
          const li = document.createElement('li');
          li.className = todo.completed ? 'completed' : '';

          // Top row: checkbox, teks tugas, tombol hapus
          const topDiv = document.createElement('div');
          topDiv.className = 'todo-top';

          const checkbox = document.createElement('input');
          checkbox.type = 'checkbox';
          checkbox.checked = todo.completed;
          checkbox.addEventListener('change', () => {
            todo.completed = checkbox.checked;
            if(checkbox.checked) {
              todo.completedAt = new Date().toISOString();
            } else {
              todo.completedAt = null;
            }
            saveCategories();
            renderCategories();
          });

          const todoText = document.createElement('span');
          todoText.textContent = todo.text;
          todoText.className = 'todo-text';

          const delTodoBtn = document.createElement('button');
          delTodoBtn.textContent = 'Hapus';
          delTodoBtn.onclick = () => {
            cat.todos.splice(idx, 1);
            saveCategories();
            renderCategories();
          };

          topDiv.appendChild(checkbox);
          topDiv.appendChild(todoText);
          topDiv.appendChild(delTodoBtn);
          li.appendChild(topDiv);

          // Timestamp & info tambahan
          const timestampsDiv = document.createElement('div');
          timestampsDiv.className = 'timestamps';

          const createdAtStr = todo.createdAt ? formatDateTime(todo.createdAt) : '';
          const completedAtStr = todo.completedAt ? formatDateTime(todo.completedAt) : '';
          const tglBarangSampaiStr = todo.tglBarangSampai ? formatDateOnly(todo.tglBarangSampai) : '';
          const noSJStr = todo.noSJ || '';

          timestampsDiv.innerHTML = `
            Dibuat pada: <b>${createdAtStr || '-'}</b><br>
            ${todo.completed ? `Selesai pada: <b>${completedAtStr}</b><br>` : ''}
            ${noSJStr ? `No SJ: <b>${noSJStr}</b><br>` : ''}
            ${tglBarangSampaiStr ? `Tanggal Barang Sampai: <b>${tglBarangSampaiStr}</b>` : ''}
          `;

          li.appendChild(timestampsDiv);

          ul.appendChild(li);
        });

        section.appendChild(ul);
        categoriesContainer.appendChild(section);
      });
    }

    addCategoryBtn.onclick = () => {
      const name = categoryInput.value.trim();
      if(!name) {
        alert('Nama kategori tidak boleh kosong');
        return;
      }
      if(categories.some(c => c.name.toLowerCase() === name.toLowerCase())) {
        alert('Kategori sudah ada');
        return;
      }
      categories.push({ id: Date.now(), name, todos: [] });
      categoryInput.value = '';
      saveCategories();
      renderCategorySelect();
      renderCategories();
    };

    addTodoBtn.onclick = () => {
      const todo = todoInput.value.trim();
      const noSJ = noSJInput.value.trim();
      const tglBarangSampai = tglBarangSampaiInput.value;

      if(!todo) {
        alert('Tugas tidak boleh kosong');
        return;
      }
      if(!categorySelect.value) {
        alert('Pilih kategori terlebih dahulu');
        return;
      }

      const selectedCatId = parseInt(categorySelect.value);
      const cat = categories.find(c => c.id === selectedCatId);
      if(!cat) {
        alert('Kategori tidak ditemukan');
        return;
      }

      cat.todos.push({ 
        text: todo, 
        completed: false, 
        createdAt: new Date().toISOString(),
        completedAt: null,
        noSJ,
        tglBarangSampai
      });

      todoInput.value = '';
      noSJInput.value = '';
      tglBarangSampaiInput.value = '';

      saveCategories();
      renderCategories();
    };

    function init() {
      renderCategorySelect();
      renderCategories();

      if(categories.length === 0) {
        categories.push({ id: Date.now(), name: 'Umum', todos: [] });
        saveCategories();
        renderCategorySelect();
        renderCategories();
      }
    }

    init();
  </script>

</body>
</html>
