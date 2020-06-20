<template>
  <div class="home">
    <div>
      <b-container class="mt-3">
        <b-row class="mb-3">
          <b-col cols="6" offset="3">
            <b-button v-b-modal.modal-prevent-closing variant="success" block>Create New Post</b-button>
          </b-col>
        </b-row>
        <b-row>
          <b-col cols="4" v-for="post in posts" v-bind:key="post.id">
            <b-card
              :title="post.title"
              img-src="https://picsum.photos/600/300/?image=25"
              img-alt="Image"
              img-top
              tag="article"
              style="max-width: 20rem;"
              class="mb-2"
            >
              <b-card-text>{{post.description}}</b-card-text>
              <b-badge
                v-for="category in post.categories"
                v-bind:key="category"
                pill
                variant="secondary"
              >{{category}}</b-badge>
              <div class="mb-3"></div>
              <b-button variant="success" class="mr-5" @click="beforeEdit(post)">Edit</b-button>
              <b-button variant="outline-danger" @click="deletePost(post.id)">Delete</b-button>
            </b-card>
          </b-col>
        </b-row>
      </b-container>

      <b-modal
        id="modal-prevent-closing"
        ref="modal"
        title="Add New Post"
        @hidden="resetModal"
        @ok="handleOk"
      >
        <form ref="form" @submit.stop.prevent="handleSubmit">
          <b-form-group
            :state="title"
            label="Title"
            label-for="title-input"
            invalid-feedback="Title is required"
          >
            <b-form-input id="title-input" v-model="title" :state="titleState" required></b-form-input>
          </b-form-group>
          <b-form-group
            :state="nameState"
            label="Description"
            label-for="description-input"
            invalid-feedback="Description is required"
          >
            <b-form-input
              id="description-input"
              v-model="description"
              :state="descriptionState"
              required
            ></b-form-input>
          </b-form-group>
          <div>
            Categories:
            <span v-for="item in selected" v-bind:key="item">{{ ` ${item},` }}</span>
          </div>
          <b-form-group>
            <b-dropdown
              text="Select Categories"
              block
              split
              split-variant="outline-primary"
              variant="primary"
              class="m-2"
            >
              <b-dropdown-item v-b-modal.modal-category>Create a new category</b-dropdown-item>
              <b-dropdown-divider></b-dropdown-divider>

              <div class="ml-3">
                <b-form-checkbox
                  v-for="option in options"
                  v-model="selected"
                  :key="option.value"
                  :value="option.value"
                  name="flavour-3a"
                >{{ option.text }}</b-form-checkbox>
              </div>
            </b-dropdown>
          </b-form-group>
        </form>
        <b-modal
          id="modal-category"
          title="Add New Category"
          @hidden="resetModalCategory"
          @ok="handleOkCategory"
        >
          <form ref="form1" @submit.stop.prevent="handleSubmitCategory">
            <b-form-group
              :state="categoryState"
              label="Category"
              label-for="category-input"
              invalid-feedback="Categoy is required"
            >
              <b-form-input id="category-input" v-model="category" :state="categoryState" required></b-form-input>
            </b-form-group>
          </form>
        </b-modal>
      </b-modal>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      name: "",
      nameState: null,
      category: "",
      categoryState: null,
      title: "",
      titleState: null,
      description: "",
      descriptionState: null,
      submittedNames: [],
      selected: [],
      nexId: 4,
      onEditing: false,
      currId: -1,
      posts: [
        {
          id: 1,
          title: "Post 1",
          description: "something",
          categories: ["orange", "apple", "pineapple"]
        },
        {
          id: 2,
          title: "Post 2",
          description: "something",
          categories: ["orange", "apple", "pineapple"]
        },
        {
          id: 3,
          title: "Post 3",
          description: "something",
          categories: ["one", "two", "three"]
        }
      ],
      options: [
        { text: "Orange", value: "orange" },
        { text: "Apple", value: "apple" },
        { text: "Pineapple", value: "pineapple" },
        { text: "Grape", value: "grape" }
      ]
    };
  },
  computed: {
    currentSelected() {
      return this.selected;
    }
  },
  methods: {
    checkFormValidity() {
      const valid = this.$refs.form.checkValidity();
      this.nameState = valid;
      this.descriptionState = valid;
      this.titleState = valid;
      return valid;
    },
    resetModal() {
      this.title = "";
      this.titleState = null;
      this.description = "";
      this.descriptionState = null;
      this.currId = -1;
      this.onEditing = false;
      this.selected = [];
    },
    handleOk(bvModalEvt) {
      bvModalEvt.preventDefault();
      this.handleSubmit();
    },
    handleSubmit() {
      if (!this.checkFormValidity()) {
        return;
      }
      if (this.onEditing) this.editPost(this.currId);
      else this.addPost();
      this.$nextTick(() => {
        this.$bvModal.hide("modal-prevent-closing");
      });
    },
    addPost() {
      this.posts.push({
        id: this.nextId,
        title: this.title,
        description: this.description,
        categories: this.selected
      });
      this.nextId++;
    },
    beforeEdit(post) {
      this.currId = post.id;
      this.title = post.title;
      this.description = post.description;
      this.onEditing = true;
      this.selected = post.categories;
      this.$bvModal.show("modal-prevent-closing");
    },
    editPost(id) {
      const index = this.posts.findIndex(item => item.id === id);
      this.posts.splice(index, 1, {
        id: id,
        title: this.title,
        description: this.description,
        categories: this.selected
      });
      this.currId = -1;
      this.onEditing = false;
    },
    deletePost(id) {
      const index = this.posts.findIndex(item => item.id == id);
      this.posts.splice(index, 1);
    },
    checkFormValidityCategory() {
      const valid = this.category.trim().length != 0;
      this.categoryState = valid;
      return valid;
    },
    resetModalCategory() {
      this.category = "";
      this.categoryState = null;
    },
    handleOkCategory(bvModalEvt) {
      bvModalEvt.preventDefault();
      this.handleSubmitCategory();
    },
    handleSubmitCategory() {
      if (!this.checkFormValidityCategory()) {
        return;
      }

      this.options.push({
        text: this.category,
        value: this.category
      });
      this.selected.push(this.category);
      this.$nextTick(() => {
        this.$bvModal.hide("modal-category");
      });
    }
  }
};
</script>
