<template>
  <div class="container">
    <div class="card">
      <div class="card-header">
        <h2 class="card-title">{{ form.id ? 'Edit Article' : 'Nama & Spesifikasi Laptop' }}</h2>
      </div>
      <div class="card-body">
        <form @submit.prevent="save">
          <div class="form-group">
            <label for="title" class="form-label">Title</label>
            <input type="text" id="title" v-model="form.title" class="form-control" placeholder="Title" />
          </div>
          <div class="form-group">
            <label for="content" class="form-label">Content</label>
            <textarea id="content" v-model="form.content" class="form-control" placeholder="Content" rows="4"></textarea>
          </div>
          <button type="submit" class="btn btn-primary">Save</button>
        </form>
      </div>
    </div>

    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div class="article-content">
          <h5>{{ article.title }}</h5>
          <p>{{ article.content }}</p>
        </div>
        <div class="article-actions">
          <button @click="edit(article)" class="btn btn-warning">Edit</button>
          <button @click="remove(article.id)" class="btn btn-danger">Delete</button>
        </div>
      </li>
    </ul>

    <button @click="load" class="btn btn-secondary">Reload Articles</button>
    <p class="welcome-text">Welcome to Data Buku View</p>
  </div>
</template>

<script>
import { ref, reactive, onMounted } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : "http://localhost:3000/articles";
        const method = form.id ? "put" : "post";
        const response = await axios[method](url, {
          title: form.title,
          content: form.content,
        });

        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        form.id = null;
        form.title = "";
        form.content = "";
      } catch (error) {
        console.error("Error saving article:", error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, remove, load };
  }
}
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
  background-image: linear-gradient(120deg, #84fab0 0%, #8fd3f4 100%);
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.card {
  background: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.card-header {
  padding: 15px;
  border-bottom: 1px solid #e0e0e0;
}

.card-title {
  margin: 0;
}

.card-body {
  padding: 15px;
}

.form-group {
  margin-bottom: 15px;
}

.form-label {
  display: block;
  margin-bottom: 5px;
}

.form-control {
  width: 100%;
  padding: 10px;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
}

.btn {
  display: inline-block;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
  text-align: center;
  text-decoration: none;
  color: #ffffff;
}

.btn-primary {
  background-color: #007bff;
  border: none;
}

.btn-warning {
  background-color: #ffc107;
  border: none;
}

.btn-danger {
  background-color: #dc3545;
  border: none;
}

.btn-secondary {
  background-color: #6c757d;
  border: none;
}

.article-list {
  list-style: none;
  padding: 0;
}

.article-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  border-bottom: 1px solid #e0e0e0;
}

.article-content {
  flex: 1;
}

.article-actions {
  display: flex;
  gap: 10px;
}

.welcome-text {
  text-align: center;
  margin-top: 20px;
  font-size: 1.2em;
  color: #333333;
}
</style>
