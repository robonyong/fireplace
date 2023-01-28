<script>
  export let color = null;
</script>

<div
  class="block"
  class:background={color === null}
  class:log-dark={color === "log-dark"}
  class:log-light={color === "log-light"}
  class:coal-red={color === "coal-red"}
  class:coal-orange={color === "coal-orange"}
  class:coal-yellow={color === "coal-yellow"}
  class:flame-red={color === "flame-red"}
  class:flame-orange={color === "flame-orange"}
  class:flame-yellow={color === "flame-yellow"}
  class:ember1={color === "ember1"}
  class:ember2={color === "ember2"}
  class:ember3={color === "ember3"}
/>

<style>
  .block {
    height: 8px;
    width: 8px;
  }
  .background {
    background-color: black;
  }
  .log-dark {
    background-color: rgb(78, 53, 36);
  }
  .log-light {
    background-color: rgb(92, 44, 38);
  }
  .coal-red {
    background-color: rgb(155, 0, 0);
  }
  .coal-yellow {
    background-color: rgb(255, 190, 44);
  }
  .flame-orange {
    background-color: rgb(232, 141, 14);
  }
  .flame-yellow {
    background-color: rgb(255, 184, 28);
  }
  .flame-red {
    background-color: rgb(153, 0, 20);
  }
  .ember1 {
    background-color: rgb(255, 87, 0);
  }
  .ember2 {
    background-color: rgb(219, 108, 0);
  }
  .ember3 {
    background-color: rgb(255, 44, 44);
  }
</style>
