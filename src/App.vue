<template>
  <div id="app">
    <FunnelEditor
      :background="background"
      :grid-size="gridSize"
      :draw-grid="drawGrid"
      :backgroundone=" backgroundone"
      @init="setupGraph"
    />
  </div>
</template>

<script>
import FunnelEditor from "@/components/FunnelEditor";
import _ from "lodash";
import $ from "jquery/dist/jquery";

export default {
  name: "App",

  components: {
    FunnelEditor
  },

  data() {
    return {
      background: {
        color: "rgba(0, 255, 0, 0.3)"
      },
      backgroundone: {
        color: "green"
      },
      gridSize: 1,
      drawGrid: {
        name: "mesh"
      }
    };
  },

  methods: {
    setupGraph(graphTwo) {
      // ----------------------------------------------

      var cythia = this;
      this.$joint.shapes.html = {};
      this.$joint.shapes.html.Element = this.$joint.shapes.basic.Rect.extend({
        defaults: this.$joint.util.util.deepSupplement(
          {
            type: "html.Element",
            attrs: {
              rect: { stroke: "none", "fill-opacity": 0 }
            }
          },
          this.$joint.shapes.basic.Rect.prototype.defaults
        )
      });

      this.$joint.shapes.html.ElementView = this.$joint.dia.ElementView.extend({
        template: [
          '<div class="main-object">',
          "<h3>Testing Component</h3>",
          "</div>"
        ].join(""),

        initialize: function() {
          console.log("this b4 bindAll", this);
          _.bindAll(this, "updateBox"); // update the ppt of the function to the this object (ElementView)
          cythia.$joint.dia.ElementView.prototype.initialize.apply(
            this,
            arguments
          );

          console.log("this after bindAll", this);

          this.$box = $(_.template(this.template)());
          console.log("this.model", this.model);
          console.log("this.$box", this.$box);

          // Update the box position whenever the underlying model changes.
          this.model.on("change", this.updateBox, this);

          this.updateBox();
        },
        render: function() {
          cythia.$joint.dia.ElementView.prototype.render.apply(this, arguments);
          console.log("paper inside render", this.paper);
          this.paper.$el.prepend(this.$box);
          console.log(this.paper.$el.prepend(this.$box));
          this.updateBox();
          return this;
        },
        updateBox: function() {
          // Set the position and dimension of the box so that it covers the JointJS element.
          var bbox = this.model.getBBox(); // bbox now represent the model rect
          console.log("bbox", bbox);
          this.$box.css({
            width: bbox.width,
            height: bbox.height,
            left: bbox.x,
            top: bbox.y,
            transform: "rotate(" + (this.model.get("angle") || 0) + "deg)"
          });
        }
      });
      // ----------------------------------------------
      // new decorated image --------------------------
      this.$joint.shapes.basic.DecoratedRect = this.$joint.shapes.basic.Generic.extend(
        {
          markup:
            '<g class="rotatable"><g class="scalable"><rect/></g><image/><text/></g>',

          defaults: this.$joint.util.util.deepSupplement(
            {
              type: "basic.DecoratedRect",
              size: { width: 100, height: 60 },
              attrs: {
                rect: {
                  fill: "#FFFFFF",
                  stroke: "black",
                  width: 100,
                  height: 60
                },
                text: {
                  "font-size": 14,
                  text: "",
                  "ref-x": 0.5,
                  "ref-y": 0.5,
                  ref: "rect",
                  "y-alignment": "middle",
                  "x-alignment": "middle",
                  fill: "black"
                },
                image: {
                  "ref-x": 2,
                  "ref-y": 2,
                  ref: "rect",
                  width: 16,
                  height: 16
                }
              }
            },
            this.$joint.shapes.basic.Generic.prototype.defaults
          )
        }
      );

      // new custom Element -----------------------------
      var CustomElement = this.$joint.dia.Element.define(
        "examples.CustomElement",
        {
          attrs: {
            body: {
              refWidth: "100%",
              refHeight: "100%",
              strokeWidth: 2,
              stroke: "black",
              fill: "white"
            },
            label: {
              textVerticalAnchor: "middle",
              textAnchor: "middle",
              refX: "50%",
              refY: "50%",
              fontSize: 14,
              fill: "black"
            },
            button: {
              cursor: "pointer",
              ref: "buttonLabel",
              refWidth: "150%",
              refHeight: "150%",
              refX: "-25%",
              refY: "-25%"
            },
            buttonLabel: {
              pointerEvents: "none",
              refX: "6%",
              refY: "-28%",
              textAnchor: "middle",
              textVerticalAnchor: "middle"
            }
          }
        },
        {
          markup: [
            {
              tagName: "rect",
              selector: "body"
            },
            {
              tagName: "text",
              selector: "label"
            },
            {
              tagName: "rect",
              selector: "button"
            },
            {
              tagName: "text",
              selector: "buttonLabel"
            }
          ]
        }
      );

      var element = new CustomElement();
      element.position(190, 200);
      element.resize(100, 40);
      element.attr({
        label: {
          pointerEvents: "none",
          visibility: "visible",
          text: "Element"
        },
        body: {
          cursor: "default",
          visibility: "visible"
        },
        button: {
          event: "element:button:pointerdown",
          fill: "white",
          stroke: "black",
          strokeWidth: 0
        },
        buttonLabel: {
          text: "X", // fullwidth underscore
          fill: "black",
          fontSize: 12,
          fontWeight: "bold"
        }
      });

      var element1 = element.clone();
      element1.position(190, 100);
      element1.resize(100, 40);
      // ------------------------------

      const rect = new this.$joint.shapes.standard.Rectangle();
      rect.position(190, 250);
      rect.resize(100, 40);
      rect.attr({
        body: {
          fill: "blue",
          magnet: true
        },
        label: {
          text: "Hello",
          fill: "white"
        }
      });

      const rect2 = rect.clone();
      rect2.translate(180, 0);
      rect2.attr("label/text", "World!");

      var em1 = new this.$joint.shapes.html.Element({
        position: { x: 80, y: 80 },
        size: { width: 170, height: 100 }
      });

      graphTwo.addCells([rect, rect2, element, element1, em1]);
      graphTwo.set("graphCustomProperty", true);
      graphTwo.set("graphExportTime", Date.now());
      var jsonObject = graphTwo.toJSON();
      console.log("no of object", jsonObject);
    }

    //methods end
  }
};
</script>

<style>
</style>
