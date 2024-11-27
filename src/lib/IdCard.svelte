<script>
  export let id_image =
    "https://www.showbizjunkies.com/wp-content/uploads/2021/05/next-level-chef-gordon-ramsay.jpg";
  export let name = "Gordon Ramsay";
  export let subtitle = "Chef";
</script>

<div class="card">
  <div class="id_image_container">
    <img class="id_image" src={id_image} alt={name} />
  </div>
  <div>
    <div class="name">{name}</div>
    <div class="subtitle">{subtitle}</div>
  </div>
</div>

<style>
  .card {
    background-color: var(--background);
    border: 1px solid var(--border);
    border-radius: 10px;
    padding: 10px;
    display: flex;
  }
  .id_image_container {
    overflow: hidden;
    height: 100px;
    width: 100px;
    border-radius: 10px;
  }
  .id_image {
    min-width: 100px;
    min-height: 100px;
  }
  .name {
    font-weight: bold;
    margin-left: 10px;
  }
  .subtitle {
    color: var(--text-muted);
    margin-left: 10px;
  }
</style>
