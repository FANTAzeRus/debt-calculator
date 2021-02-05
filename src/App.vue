<template>
  <DebtCalculator />
</template>

<script>
import DebtCalculator from "./components/Calculator";
import "./main.scss";

export default {
  name: "App",
  components: { DebtCalculator },
};
</script>