<template>
<div class="admin">
    <h1>The Admin Page!</h1>
    <div class="heading">
      <h2>Add an Item</h2>
    </div>
    <div class="add">
      <div class="flex1">
        <div class="form">
          <input v-model="title" placeholder="Title">
          <br>
          <input v-model="textarea" placeholder="Description" type="textarea">
          <br>
          <input v-model="price" placeholder="Price" >
          <br>
          <input v-model="size" placeholder="Size">
          <br>
          <input style="width: 239px" type="file" name="photo" @change="fileChanged">
          <br>
          <button @click="upload">Upload</button>
        </div>
      </div>
      <div class="flex1">
        <div class="upload" v-if="addItem">
          <h2>{{addItem.title}}</h2>
          <h2>{{addItem.textarea}}</h2>
          <h2>{{addItem.price}}</h2>
          <h2>{{addItem.size}}</h2>
          <img :src="addItem.path" />
        </div>
      </div>
    </div>

    <div class="heading">
    <h2>Edit/Delete an Item</h2>
    </div>
    <div class="edit">
      <div class="flex1">
        <div class="form">
          <input v-model="findTitle" placeholder="Search">

          <div class="suggestions" v-if="suggestions.length > 0">
            <div class="suggestion" v-for="s in suggestions" :key="s.id" @click="selectItem(s)">{{s.title}}
            </div>
          </div>
        </div>
      </div>
      <div class="flex1">
        <div class="upload" v-if="findItem">
          <input v-model="findItem.title" placeholder="Title" class = "different_button">
          <p></p>
          <input v-model="findItem.textarea" placeholder="Description" type="textarea" class = "different_button">
          <p></p>
          <input v-model="findItem.price" placeholder="Price" class = "different_button">
          <p></p>
          <input v-model="findItem.size" placeholder="Size" class = "different_button">
          <p></p>
          <img :src="findItem.path" />
        </div>
      </div>
      <div class="flex1">
        <div class="actions" v-if="findItem">
          <button @click="deleteItem(findItem)" class="different_button">Delete</button>
          <button @click="editItem(findItem)" class="different_button">Edit</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@media only screen and (max-width: 500px) {
  .upload {
    margin-left: 100px !important;
    padding-top:10px !important;
  }
  .edit .add{
    /* justify-content: center !important; */
  }
  .form {
    margin: 0px !important;
  }
  .actions {
    margin-left: 100px !important;
  }
}
.image h2 {
  font-style: italic;
  font-size: 1em;
}

.heading {
  display: flex;
  margin-bottom: 20px;
  margin-top: 20px;
  padding-left: 100px;

}

.heading h2 {
  margin-top: 8px;
  margin-left: 10px;
  font-size: 25px;
  font-family: 'Montserrat', sans-serif;
  font-variant: small-caps;
  color: gray;
}

.add,
.edit {
  display: flex;
  flex-wrap:wrap;
  flex-direction: row;
}

.circle {
  border-radius: 50%;
  width: 18px;
  height: 18px;
  padding: 8px;
  background: #333;
  color: #fff;
  text-align: center
}

.flex1 {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
}

/* Form */
input,
textarea,
select,
button {
  font-family: 'Montserrat', sans-serif;
  font-size: 1em;
  margin-left:100px;
}
.upload {
  margin-left: 100px;
}
.different_button {
  margin-left:0px !important;
}

.add > .flex1 > .form > input{
  margin-top: 10px;
}


/* Uploaded images */
.upload h2 {
  margin: 0px;
  font-size: 15px;
  font-family: 'Montserrat', sans-serif;
  color: gray;
  padding:2px;
}

.upload img {
  max-width: 300px;
}

/* Suggestions */
.suggestions {
  width: 200px;
  border: 1px solid #ccc;
  margin-left:100px;
}

.suggestion {
  min-height: 20px;
}

.suggestion:hover {
  background-color: #5BDEFF;
  color: #fff;
}

 h1{
  padding-left: 100px !important;
  color: gray;
  font-family: "Montserrat", sans-serif !important;
  font-variant: small-caps;
}


</style>


<script>
import axios from 'axios';

export default {
  name: 'Admin',
  data() {
    return {
      title: "",
      file: null,
      addItem: null,
      items: [],
      findTitle: "",
      findItem: null,
      textarea: "",
      price: "",
      size: "",
    }
  },
  computed: {
    suggestions() {
      let items = this.items.filter(item => item.title.toLowerCase().startsWith(this.findTitle.toLowerCase()));
      return items.sort((a, b) => a.title > b.title);
    }
  },
  created() {
    this.getItems();
  },
  methods: {
    fileChanged(event) {
      this.file = event.target.files[0]
      console.log(this.file)
    },
    async upload() {
      try {
        console.log("uploading")
        const formData = new FormData();
        formData.append('photo', this.file, this.file.name)
        let r1 = await axios.post('/api/photos', formData);
        let r2 = await axios.post('/api/items', {
          title: this.title,
          textarea: this.textarea,
          price: this.price,
          size : this.size,
          path: r1.data.path
        });
        this.addItem = r2.data;
        console.log(this.addItem)
      } catch (error) {
        console.log(error);
      }
    },
    async getItems() {
      try {
        let response = await axios.get("/api/items");
        this.items = response.data;
        return true;
      } catch (error) {
        console.log(error);
      }
    },
    selectItem(item) {
      this.findTitle = "";
      this.findItem = item;
    },
    async deleteItem(item) {
      try {
        await axios.delete("/api/items/" + item._id);
        this.findItem = null;
        this.getItems();
        return true;
      } catch (error) {
        console.log(error);
      }
    },
    async editItem(item) {
      try {
        await axios.put("/api/items/" + item._id, {
          title: this.findItem.title,
          textarea: this.findItem.textarea,
          price: this.findItem.price,
          size: this.findItem.size,
        });
        this.findItem = null;
        this.getItems();
        return true;
      } catch (error) {
        console.log(error);
      }
    },



  },

}
</script>
