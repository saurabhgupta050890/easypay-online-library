<template>
  <div class="container">
    <section>
      <div class="block">
        <b-radio v-model="searchOn" native-value="ISBN">ISBN</b-radio>
        <b-radio v-model="searchOn" native-value="name">Book Name</b-radio>
      </div>

      <b-field label="Book Name or ISBN:" :type="type" :message="message">
        <b-input v-model="name"></b-input>
      </b-field>

      <button class="button is-primary" @click="searchBook">Search</button>
    </section>
    <section v-if="data.length > 0" style="padding-top: 15px;">
      <div class="list">
        <div
          style="cursor: pointer;"
          class="card-content"
          v-for="book in data"
          :key="book.identifiers.isbn_10[0]"
          @click="showBookDetails(book)"
        >
          <div class="media">
            <div class="media-left">
              <figure class="image">
                <img :src="book.cover.small" alt="book">
              </figure>
            </div>
            <div class="media-content">
              <p class="title is-4">{{ book.title }}</p>
              <p class="subtitle is-6">{{ book.subtitle || book.by_statement}}</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <b-modal :active.sync="isBookModalActive" :width="640" scroll="keep">
      <div class="card" v-if="selected">
        <div class="card-image">
          <figure class="image is-4by3">
            <img :src="selected.cover.large" alt="Image">
          </figure>
        </div>
        <div class="card-content">
          <div class="media">
            <div class="media-left">
              <figure class="image is-48x48">
                <img :src="selected.cover.small" alt="Image">
              </figure>
            </div>
            <div class="media-content">
              <p class="title is-4">{{ selected.title }}</p>
              <p class="subtitle is-6">{{ selected.subtitle || selected.by_statement}}</p>
            </div>
          </div>

          <div class="content">
            <div style="padding-top: 15px;">
              Published By:
              <span v-for="p in selected.publishers" :key="p.name">{{p.name}}</span>
            </div>
            <br>
            <div style="padding-top: 5px;">
              Authors:
              <span v-for="a in selected.authors" :key="a.name">{{a.name}}</span>
              <p>{{selected.notes}}</p>
            </div>
          </div>
        </div>
      </div>
    </b-modal>
  </div>
</template>

<script>
// @ is an alias to /src

export default {
  name: "home",
  data() {
    return {
      searchOn: "ISBN",
      data: [],
      name: "",
      type: "",
      message: "",
      selected: null,
      isBookModalActive: false
    };
  },
  methods: {
    searchBook() {
      this.message = "";
      this.type = "";
      if (this.name.length <= 4) {
        this.data = [];
        this.message = "Atleast 4 characters required";
        this.type = "is-danger";
        return;
      }

      let url;
      if (this.searchOn === "ISBN") {
        url = `https://openlibrary.org/api/books?bibkeys=ISBN:${
          this.name
        }&format=json&jscmd=data`;
        if (!/\d+/g.test(this.name)) {
          this.message = "Only numbers are allowed for ISBN";
          this.type = "is-danger";
        }
      } else {
        url = `http://openlibrary.org/search.json?title=${this.name}`;
      }

      fetch(url)
        .then(res => res.json())
        .then(book => {
          if (Object.keys(book).length > 0) {
            if (this.searchOn === "ISBN") {
              let isbn = `ISBN:${this.name}`;
              this.data = [book[isbn]];
            } else {
              if (book.numFound > 0) {
                // Fetching first 100 search results
                let docs = book.docs.slice(0, 100);
                this.data = docs
                  .filter(
                    doc => doc.cover_i && doc.author_name && doc.publisher
                  )
                  .map(doc => ({
                    title: doc.title,
                    cover: {
                      small: `http://covers.openlibrary.org/b/id/${
                        doc.cover_i
                      }-S.jpg`,
                      large: `http://covers.openlibrary.org/b/id/${
                        doc.cover_i
                      }-L.jpg`
                    },
                    authors: doc.author_name.map(n => ({ name: n })),
                    publishers: doc.publisher.map(n => ({ name: n })),
                    identifiers: {
                      isbn_10: [doc.cover_i]
                    }
                  }));
              }
            }
          } else {
            this.data = [];
          }
        });
    },
    showBookDetails(book) {
      this.selected = book;
      this.isBookModalActive = true;
    }
  }
};
</script>
