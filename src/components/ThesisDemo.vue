<template>
  <MDBCard v-for="(rating, index) in ratings" :key="index" style="margin: 1em; font-size: 20pt;">
    <MDBCardBody>
      <span v-if="rating.value === 'http://schema.org/Excellent'">🤩 </span>
      <span v-if="rating.value === 'http://schema.org/Amazing'">🚀 </span>
      <span v-if="rating.value === 'http://schema.org/Awesome'">🎉 </span>
      {{ rating.explanation }}
    </MDBCardBody>
  </MDBCard>
</template>

<script>
import {QueryEngine} from "@comunica/query-sparql";
import {MDBCard, MDBCardBody} from "mdb-vue-ui-kit";

export default {
  name: "ThesisDemo",
  components: {
    MDBCard,
    MDBCardBody,
  },
  data() {
    return {
      engine: new QueryEngine(),
      ratings: [],
    }
  },
  async created() {
    // Execute every 3 seconds
    setInterval(async () => {
      await this.queryResource();
    }, 3000);
  },
  methods: {
    async queryResource() {
      const data = await fetch('https://solid.smessie.com/thesis/forms/thesis-rating.ttl', {
        cors: "cors",
      }).then((response) => response.text());

      const bindings = await (await this.engine.queryBindings(`
        PREFIX schema: <http://schema.org/>
        SELECT ?value ?explanation WHERE {
            ?s a schema:Rating ;
                schema:ratingValue ?value ;
                schema:ratingExplanation ?explanation .
        }
      `, {
            sources: [{
              type: 'stringSource',
              value: data,
              mediaType: 'text/turtle',
            }]
          }
      )).toArray();

      this.ratings = bindings.map((binding) => {
        return {
          value: binding.get('value').value,
          explanation: binding.get('explanation').value,
        }
      }).reverse();
    }
  },
}
</script>

<style scoped>

</style>
