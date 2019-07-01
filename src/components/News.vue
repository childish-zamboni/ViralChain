<template>
  <div>
<modal>
  <div class="modal-mask">
  <div class="modal-wrapper">
    <div class="modal-container">
      <div class="modal-header">
        {{title}} by {{author}}
      </div>

      <div class="modal-body">
        {{body}}
      </div>

      <div class="modal-footer">
          <button class="modal-default-button" @click="showModal = false">
            Close
          </button>
      </div>
    </div>
  </div>
</div>
</modal>
</div>
</template>

<script>
export default {
  name: 'News',
  props: ['title', 'body', 'author'],
}
</script>

<style>

.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, .5);
  display: table;
  transition: opacity .3s ease;
}

.modal-wrapper {
  display: table-cell;
  vertical-align: middle;
  color: black!important;
}

.modal-container {
  width: 50%;
  margin: 0px auto;
  padding: 20px 30px;
  background-color: #fff;
  border-radius: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
  transition: all .3s ease;
  font-family: Helvetica, Arial, sans-serif;
}

.modal-header h3 {
  margin-top: 0;
  color: #42b983;
}

.modal-body {
  margin: 20px 0;
}

.modal-default-button {
  float: right;
}
</style>
