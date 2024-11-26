<div class="card">
  <div class="id_image_container">
    <img
      class="id_image"
      src="https://www.washingtonpost.com/resizer/KkRVfO2fP56KislxAI6qlYDPFIE=/1440x0/smart/arc-anglerfish-washpost-prod-washpost.s3.amazonaws.com/public/3NLTPJSHHQY2PPQBQ6VGZVPV3M.jpg"
      alt="Donald Trump"
    />
  </div>
  <div>
    <div class="name">Donald Trump</div>
    <div class="subtitle">President of the United States</div>
  </div>
</div>

<style>
  .card {
    background-color: var(--background);
    border: 1px solid rgb(97, 97, 97);
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
