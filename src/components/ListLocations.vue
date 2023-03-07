<template>
  <div>
    <div class="row">
      <div class="col">
        <q-list
          bordered
          separator
        >
          <q-item
            clickable
            v-ripple
            v-for="(location, index) in locations"
            :key="index"
            @click="$emit('getLocation', location)"
          >
            <q-item-section>
              <q-item-label>{{ location.address }}</q-item-label>
              <q-item-label caption>coords: {{ location.coords }}</q-item-label>
            </q-item-section>
          </q-item>

        </q-list>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: [
    'locations',
    'selectedLocation'
  ],
}
</script>

<style>
</style>