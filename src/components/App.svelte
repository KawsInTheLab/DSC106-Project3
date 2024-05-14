<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let svg;
  let projection;
  let pathGenerator;
  let data; // Define data variable

  onMount(() => {
      d3.csv("2022.csv").then((d) => {
      data = d;
      console.log(data);
      // Create SVG element
      svg = d3.select("#map");

      // Define projection and path generator
      projection = d3.geoOrthographic().rotate([0, 0]).fitSize([960, 600], { type: "Sphere" });
      pathGenerator = d3.geoPath().projection(projection);

      // Load the GeoJSON data
      d3.json("world.geojson").then(world => {
          // Merge GeoJSON features with your data
          world.features.forEach(feature => {
              const countryData = data.find(d => d.Country === feature.properties.name); // Fix 'country' to 'Country'
              feature.properties.data = countryData ? +countryData["Happiness score"] : 0; // Change value to 'Happiness score'
          });

          // Define color scale
          const colorScale = d3.scaleQuantize()
              .domain([0, d3.max(world.features, d => d.properties.data)])
              .range(["#f7fbff", "#08306b"]); // Example colors, change as needed

          // Render the map
          svg.selectAll("path")
              .data(world.features)
              .enter()
              .append("path")
              .attr("d", pathGenerator)
              .attr("fill", d => colorScale(d.properties.data));
      });
    });
  });
</script>

<style>
  /* Add any necessary styling here */
</style>

<svg id="map" width="960" height="600"></svg>
