<template>
  <div class="container-fluid FunnelEditor">
    <NavBar :title="funnel" />
    <div class="design-wrapper d-flex">
      <div class="editor-wrapper" id="editor-wrap">
        <div class="editor px-2">
          <button class="search-element">Search Elements</button>
          <div class="element-wrapper py-1" id="element-wrap" ref="jointOne"></div>
        </div>
        <div class="slide-wrapper" id="slide-wrap">
          <div class="setting-wrapper">
            <span class="settings">
              <i class="fas fa-cog"></i>
            </span>
            <span class="text-editor">Funnel Editor</span>
          </div>
          <span class="slide-close" id="slide-close" @click="closeMenu">
            <i class="fas fa-chevron-left"></i>
          </span>
        </div>
        <div class="font-wrapper">
          <i class="fas fa-chart-line"></i>
          <i class="fas fa-cloud-upload-alt"></i>
          <i class="fas fa-save"></i>
        </div>
      </div>
      <div class="main-wrapper" id="main-wrap" ref="joint"></div>
    </div>
  </div>
</template>
<script>
import NavBar from "@/components/NavBar";

export default {
  name: "funneleditor",
  components: {
    NavBar
  },
  props: {
    gridSize: {
      type: Number,
      default: 1
    },
    drawGrid: {
      type: [Object, Boolean],
      default: false
    },
    background: {
      type: [Object, Boolean],
      default: false
    },
    backgroundone: {
      type: [Object, Boolean],
      default: false
    },
    readonly: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      funnel: {
        t1: "FUNNEL",
        t2: "Builder",
        icontext: "F"
      },
      editor: {
        eimage: "/elementimage.png",
        items: 8
      },
      name: "",
      graph: null,
      graphTwo: null,
      paper: null,
      paperOne: null,
      flyGraph: null,
      flyShape: null,
      pos: null,
      offset: null
    };
  },
  methods: {
    closeMenu() {
      let slideWrapper = document.getElementById("slide-wrap");
      let editorWrapper = document.getElementById("editor-wrap");
      let slideClose = document.querySelector(".slide-close .fas");
      let mainWrapper = document.getElementById("main-wrap");
      console.log(slideClose);

      if (slideWrapper.style.marginLeft === "-335px") {
        slideWrapper.style.marginLeft = "0";
        editorWrapper.style.marginLeft = "0";
        mainWrapper.style.marginLeft = "335";
        slideClose.classList.remove("fa-chevron-right");
        slideClose.classList.add("fa-chevron-left");
      } else {
        slideWrapper.style.marginLeft = "-335px";
        editorWrapper.style.marginLeft = "-335px";
        mainWrapper.style.marginLeft = "0";
        slideClose.classList.remove("fa-chevron-left");
        slideClose.classList.add("fa-chevron-right");
      }
    }
  },

  created() {
    this.name = this.$options.name;
    var standard = this.$joint.shapes;

    this.graph = new this.$joint.dia.Graph([], {
      cellNamespace: { standard }
    });

    // graph two created
    this.graphTwo = new this.$joint.dia.Graph([], {
      cellNamespace: { standard }
    });
  },

  mounted() {
    /*eslint-env jquery*/

    this.paper = new this.$joint.dia.Paper({
      el: this.$refs.joint,
      // cellViewNamespace: this.$joint.shapes,
      model: this.graph,
      width: $("main-wrap").width(),
      height: $("main-wrap").height(),
      gridSize: this.gridSize,
      drawGrid: this.drawGrid,
      background: this.background,
      defaultLink: new this.$joint.dia.Link({
        //Code alteration to mimic label functionality-adding labels:

        attrs: {
          ".marker-source": {
            d: "M 10 0 L 0 5 L 10 10 z",
            transform: "scale(0.001)"
          }, // scale(0)(0.001)' }, // scale(0) fails in Firefox
          ".marker-target": { d: "M 10 0 L 0 5 L 10 10 z" },
          ".connection": { stroke: "red" }
        },
        router: { name: "manhattan" },
        labels: [{ position: 0.5, attrs: { text: { text: "Name" } } }]
      })
    });

    this.paperOne = new this.$joint.dia.Paper({
      el: this.$refs.jointOne,
      // cellViewNamespace: this.$joint.shapes,
      model: this.graphTwo,
      width: $("element-wrap").width(),
      height: $("element-wrap").height(),
      gridSize: this.gridSize,
      drawGrid: this.drawGrid,
      background: $("element-wrap").backgroundone,
      interactive: false
    });

    //for graph two
    this.$emit("init", this.graphTwo);

    this.paperOne.on("cell:pointerdown", (cellView, e, x, y) => {
      $("body").append(
        `<div id="flyPaper" 
        style="background: transparent;position:fixed;z-index:100;opacity:0.7;pointer-event:none">
        </div>`
      );

      this.flyGraph = new this.$joint.dia.Graph();
      this.flyPaper = new this.$joint.dia.Paper({
        el: $("#flyPaper"),
        model: this.flyGraph,
        interactive: false
      });

      // ------------------------------------
      this.flyShape = cellView.model.clone();
      this.pos = cellView.model.position();
      this.offset = {
        x: x - this.pos.x,
        y: y - this.pos.y
      };

      this.flyShape.position(0, 0);
      this.flyGraph.addCell(this.flyShape);

      $("#flyPaper").offset({
        left: e.pageX - this.offset.x,
        top: e.pageY - this.offset.y
      });

      $("body").on("mousemove.fly", evt => {
        $("#flyPaper").offset({
          left: evt.pageX - this.offset.x,
          top: evt.pageY - this.offset.y
        });
      });

      // -----------------------
      $("body").on("mouseup.fly", evt => {
        var x = evt.pageX,
          y = evt.pageY,
          target = this.paper.$el.offset();

        if (
          x > target.left &&
          x < target.left + this.paper.$el.width() &&
          y > target.top &&
          y < target.top + this.paper.$el.height()
        ) {
          this.s = this.flyShape.clone();
          this.s.position(
            x - target.left - this.offset.x,
            y - target.top - this.offset.y
          );
          this.graph.addCell(this.s);
        }
        $("body")
          .off("mousemove.fly")
          .off("mouseup.fly");
        this.flyShape.remove();
        $("#flyPaper").remove();
      });
    });

    // new methods to delete

    this.paper.on("element:button:pointerdown", function(elementView, evt) {
      evt.stopPropagation(); // stop any further actions with the element view (e.g. dragging)

      var model = elementView.model;
      model.remove();
    });
  }
};
</script>
<style scoped>
.container-fluid {
  padding: 0;
  height: 100vh;
}
.design-wrapper {
  background: #f0f3fe;
  height: calc(100vh - 50px);
}

.editor-wrapper {
  background: #e6e9f0;
  width: 335px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  /* position: fixed; */
  height: 100%;
  transition: 0.5s;
}

.search-element {
  background: transparent;
  width: 308px;
  padding: 8px 8px;
  margin-top: 3.3rem;
  border: none;
  border: 1px solid black;
  display: flex;
  justify-content: start;
  letter-spacing: 2px;
}

.editor {
  height: 700px;
  display: flex;
  flex-direction: column;
}
/* first div for canvas */
.element-wrapper {
  margin-top: 5px;
  height: 83%;
  /* overflow-x: hidden; */
}

.element-wrap {
  overflow-x: auto;
}

/* first div for canvas */
.main-wrapper {
  transition: 0.5s;
  width: 100%;
  height: 100%;
}
/* --slide button wrapper-- */
.slide-wrapper {
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: fixed;
  left: 0;
  background: blue;
  width: 365px;
  padding-left: 20px;
  padding-right: 8px;
  border-top-right-radius: 15px;
  border-bottom-right-radius: 15px;
  transition: 0.5s;
  z-index: 1;
}

.setting-wrapper {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 70%;
  color: white;
  padding: 5px 0;
}

.text-editor {
  font-size: 25px;
  letter-spacing: 3px;
}

.slide-wrapper .fas {
  color: white;
  font-size: 1.7rem;
}

.slide-button .fas {
  font-size: 2.1rem;
}

.font-wrapper {
  background: blue;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 25px;
}

.font-wrapper i {
  color: white;
  font-size: 19px;
}
</style>