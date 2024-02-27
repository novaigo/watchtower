<template>
  <div>
    <TheHeader />
    <section>
      <Nuxt />
    </section>
  </div>
</template>

<script>
import TheHeader from "~/components/TheHeader";
export default {
  name: "DefaultLayout",
  components: {
    TheHeader,
  },
};
</script>
