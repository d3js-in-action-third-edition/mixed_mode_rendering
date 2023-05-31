<script>
  import { forceSimulation, forceX, forceY, forceCollide } from "d3-force";
  import { scaleOrdinal } from "d3-scale";
  import { subjects } from "../utils/subjects";
  import { onMount } from "svelte";

  export let paintingAreaScale;
  export let paintingDefaultRadius;
  // export let paintings;
  export let monthScale;
  export let radius;
  // export let isTooltipVisible = false;
  // export let tooltipMeta = {};
  // export let isPeriodSelected;
  // export let selectedPeriod;

  export let width;
  export let height;
  export let paintings;
  export let yearsTranslations;

  const colorScale = scaleOrdinal()
    .domain(subjects.map((d) => d.subject))
    .range(subjects.map((d) => d.color));

  let simulation = forceSimulation(paintings);
  let nodes = [];
  simulation.on("tick", () => {
    nodes = simulation.nodes();
  });

  let canvasElement;
  const handleSimulationEnd = () => {
    console.log("sim end");
    console.log("nodes", nodes);

    const context = canvasElement.getContext("2d");

    nodes.forEach((node) => {
      const yearTranslations = yearsTranslations.find(
        (y) => y.year === node.year
      );
      context.fillStyle = colorScale(node.subject);
      switch (node.medium) {
        case "oil":
          context.strokeStyle = "#FFFAFC";
          break;
        case "watercolor":
          context.strokeStyle = "#160E13";
          break;
        case "print":
          context.strokeStyle = "#BC5D9A";
          break;
      }
      context.beginPath();
      context.arc(
        node.x,
        node.y,
        node.area_cm2
          ? Math.sqrt(paintingAreaScale(node.area_cm2) / Math.PI)
          : paintingDefaultRadius,
        0,
        2 * Math.PI,
        true
      );
      context.fill();
      context.stroke();
    });
  };

  $: {
    simulation
      .force(
        "x",
        forceX((d) => {
          const translation = yearsTranslations.find(
            (y) => y.year === d.year
          ).translationX;
          return d.month !== ""
            ? translation + radius * Math.sin(monthScale(d.month))
            : translation;
        }).strength(0.5)
      )
      .force(
        "y",
        forceY((d) => {
          const translation = yearsTranslations.find(
            (y) => y.year === d.year
          ).translationY;
          return d.month !== ""
            ? translation - radius * Math.cos(monthScale(d.month))
            : translation;
        }).strength(0.5)
      )
      .force(
        "collide",
        forceCollide()
          .radius((d) =>
            d.width_cm === null && d.height_cm === null
              ? paintingDefaultRadius + 1
              : Math.sqrt(paintingAreaScale(d.area_cm2) / Math.PI) + 1
          )
          .strength(1)
      )
      .alpha(0.5)
      .alphaDecay(0.1)
      .on("end", handleSimulationEnd);
  }

  // const handleMouseEnter = (e, node) => {
  //   isTooltipVisible = true;
  //   tooltipMeta = {
  //     x: e.offsetX,
  //     y: e.offsetY,
  //     screenY: e.clientY,
  //     url: node.imageLink,
  //     title: node.title,
  //     createdIn: node.created_in,
  //     date: node.month !== "" ? `${node.month} ${node.year}` : node.year,
  //     medium: node.medium,
  //     currentLocation: node.current_location,
  //     width: node.width_cm,
  //     height: node.height_cm,
  //     subject: node.subject,
  //   };
  // };
  // const handleMouseLeave = () => {
  //   isTooltipVisible = false;
  // };
</script>

<!-- {#each nodes as node}
  <circle
    class:watercolor={node.medium === "watercolor"}
    class:print={node.medium === "print"}
    class:lessen={isPeriodSelected && node.period !== selectedPeriod}
    cx={node.x}
    cy={node.y}
    r={node.area_cm2
      ? Math.sqrt(paintingAreaScale(node.area_cm2) / Math.PI)
      : paintingDefaultRadius}
    fill={colorScale(node.subject)}
    on:mouseover={(e) => handleMouseEnter(e, node)}
    on:focus={(e) => handleMouseEnter(e, node)}
    on:mouseleave={() => handleMouseLeave()}
  />
{/each} -->

<canvas {width} {height} bind:this={canvasElement} />

<style lang="scss">
  // circle {
  //   cursor: pointer;
  //   stroke: $white;
  //   transition: opacity 100ms ease;
  //   &.watercolor {
  //     stroke: $text;
  //   }
  //   &.print {
  //     stroke: $secondaryPale;
  //   }
  //   &.lessen {
  //     fill-opacity: 0.1;
  //     stroke-opacity: 0.1;
  //   }
  // }
  canvas {
    position: absolute;
    top: 0;
    left: 0;
  }
</style>
