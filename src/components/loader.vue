<template>
  <div class="loader" @change="change" @dragover="dragover" @drop="drop">
    <p>
      Drop image here or
      <label class="browse"
        >browse...
        <input
          id="file"
          class="sr-only"
          type="file"
          accept="image/*"
          multiple
        />
      </label>
    </p>
    <ul class="loadedFiles">
      <li><button v-on:click="prev">prev</button></li>
      <li><button v-on:click="next">next</button></li>
      <li
        v-for="(file, index) in storedfiles"
        v-on:click="myupdate(file)"
        v-bind:key="index + '-file'"
      >
        <button>{{ index }}</button>
      </li>
    </ul>
  </div>
</template>

<script>
const URL = window.URL || window.webkitURL;

export default {
  name: "Loader",

  props: {
    data: {
      type: Object,

      default: () => ({}),
    },
  },
  data: function () {
    return {
      storedfiles: [],
      loadedItem: 0,
    };
  },
  mounted() {
    window.addEventListener(
      "keydown",
      (this.onKeydown = this.keydown.bind(this))
    );
  },
  methods: {
    keydown(e){
      console.log(e.key)
      if(e.key == 6)
        this.next();
      if(e.key == 4)
        this.prev();
    },
    prev() {
      this.loadedItem--;
      this.nextprev()
    },
    next() {
      this.loadedItem++;
      this.nextprev()
    },
    nextprev(){
      this.myupdate(this.storedfiles[this.loadedItem])
    },
    read(files) {
      return new Promise((resolve, reject) => {
        if (!files || files.length === 0) {
          resolve();
          return;
        }
        const file = files[0];
        this.saveFiles(files, this);
        if (/^image\/\w+$/.test(file.type)) {
          if (URL) {
            resolve(this.createFilesObj(file));
          } else {
            reject(new Error("Your browser is not supported."));
          }
        } else {
          reject(new Error("Please choose an image file."));
        }
      });
    },
    saveFiles(f, that) {
      for (var i = 0; i < f.length; i++) {
        console.log(that.files);
        this.storedfiles.push(this.createFilesObj(f[i]));
      }
    },
    createFilesObj(file) {
      return {
        loaded: true,
        name: file.name,
        type: file.type,
        url: URL.createObjectURL(file),
      };
    },
    change({ target }) {
      this.read(target.files)
        .then((data) => {
          target.value = "";
          this.update(data);
        })
        .catch((e) => {
          target.value = "";
          this.alert(e);
        });
    },

    dragover(e) {
      e.preventDefault();
    },

    drop(e) {
      e.preventDefault();
      this.read(e.dataTransfer.files)
        .then((data) => {
          this.update(data);
        })
        .catch(this.alert);
    },

    alert(e) {
      window.alert(e && e.message ? e.message : e);
    },
    myupdate(data) {
      this.$emit("reset");
      this.update(JSON.parse(JSON.stringify(data)));
    },
    update(data) {
      console.log(this.data);
      console.log("-----" + data);
      console.log(data);
      console.log("-----" + data);
      Object.assign(this.data, data);
    },
  },
};
</script>

<style scoped>
.loadedFiles {
  list-style: none;
  position: absolute;
  top: 0;
  left: 0;
  color: white;
}
.loader {
  display: table;
  height: 100%;
  overflow: hidden;
  width: 100%;
}

.loader > p {
  color: #999;
  display: table-cell;
  text-align: center;
  vertical-align: middle;
}

.browse {
  color: #0074d9;
  cursor: pointer;
  margin-left: 0.25rem;
}
.browse:hover {
  color: #08f;
  text-decoration: underline;
}
</style>
