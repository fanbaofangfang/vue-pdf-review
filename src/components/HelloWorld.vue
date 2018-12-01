<template>
  <div>
    <div class="pageButton">
      <span  @click="onPrevPage">上一页</span>
      <span  @click="onNextPage">下一页</span>
    </div>
    <canvas id="the-canvas"></canvas>
    <div>
      <span class="page">
        <span id="page_num">{{pageNum}}</span> /
        <span id="page_count">{{totalPages}}</span>
      </span>
    </div>
  </div>
</template>

<script>
import PDFJS from "pdf.js";
const PDFPATH = require("../assets/test.pdf");
export default {
  name: "HelloWorld",
  data() {
    return {
      pdfDoc: null,
      pageNum: 1,
      pageRendering: false,
      pageNumPending: null,
      scale: 1,
      totalPages: "",
    };
  },
  created() {
    const that = this;
    PDFJS.getDocument(PDFPATH).then(function(pdfDoc_) {
      that.pdfDoc = pdfDoc_;
      that.totalPages = that.pdfDoc.numPages;
      that.renderPage(that.pageNum);
    });
  },
  methods: {
    renderPage(num) {
      var canvas = document.getElementById("the-canvas"),
        ctx = canvas.getContext("2d");
      this.pageRendering = true;
      var that = this;
      this.pdfDoc.getPage(num).then(function(page) {
        var viewport = page.getViewport(that.scale);
        canvas.height = viewport.height;
        canvas.width = viewport.width;

        var renderContext = {
          canvasContext: ctx,
          viewport: viewport
        };
        var renderTask = page.render(renderContext);
        renderTask.promise.then(function() {
          that.pageRendering = false;
          if (that.pageNumPending !== null) {
            // New page rendering is pending
            that.renderPage(pageNumPending);
            that.pageNumPending = null;
          }
        });
      });
    },
    queueRenderPage(num) {
      if (this.pageRendering) {
        this.pageNumPending = num;
      } else {
        this.renderPage(num);
      }
    },
    onPrevPage() {
      if (this.pageNum <= 1) {
        return;
      }
      this.pageNum--;
      this.queueRenderPage(this.pageNum);
    },
    onNextPage() {
      if (this.pageNum >= this.pdfDoc.numPages) {
        return;
      }
      this.pageNum++;
      this.queueRenderPage(this.pageNum);
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
