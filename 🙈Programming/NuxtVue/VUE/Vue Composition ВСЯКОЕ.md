## Props Emits Declaration

```js
<script setup>
const props = defineProps(['foo'])
const emit = defineEmits(['change', 'delete'])

console.log(props.foo)
</script>

export default {
  props: ['foo'],
  setup(props) {
    // setup() receives props as the first argument.
    console.log(props.foo)
  }
}

```