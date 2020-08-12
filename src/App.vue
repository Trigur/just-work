<template>
    <pin-code v-model="pin"></pin-code>
</template>

<script>
import PinCode from '~/PinCode.vue';

export default {
    name: 'App',

    components: {
        PinCode,
    },

    data() {
        return {
            pin: '',
        };
    },
};
</script>

<style>
* {
   padding: 0;
   margin: 0;
   box-sizing: border-box;
}

body {
    height: 100vh;
}
</style>
