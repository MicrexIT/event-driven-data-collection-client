<script>
  import { getContext, onDestroy } from 'svelte'
  export let node
  const { getCyInstance } = getContext('graphSharedState')
  const randomNumber = (max = 100) => Math.round(max * Math.random())
  const cyInstance = getCyInstance()
  cyInstance.add({
    group: 'nodes',
    id: node.id,
    data: { ...node },
    position: {x: randomNumber(400), y: randomNumber(400)}
  })
  onDestroy(() => {
    cyInstance.remove(`#${node.id}`)
  })
</script>