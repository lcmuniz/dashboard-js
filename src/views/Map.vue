<template>
  <div class="flex mb-4 h-screen">
    <div class="w-2/3 bg-grey-light">
      <sui-dimmer :active="loading" inverted>
        <sui-loader content="Searching..." />
      </sui-dimmer>
      <l-map
        :zoom="zoom"
        :center="center"
        :options="mapOptions"
        style="height: 100%"
        @update:center="centerUpdate"
        @update:zoom="zoomUpdate">
        <l-tile-layer
          :url="url"
          :attribution="attribution"/>
          <vue2-leaflet-markercluster>
            <l-marker v-for="resource in resources" :key="resource.uuid" 
              :lat-lng="getLocation(resource)">
              <l-popup>
                <div>
                  <b>UUID: </b>{{ resource.uuid }}<br/>
                  <b>Description: </b>{{ resource.description}}<br/>
                  <b>Capabilities: </b>{{ resource.capabilities}}<br/>
                  <b>Latitude: </b>{{ resource.lat }}<br/>
                  <b>Longitude: </b>{{ resource.lon }}<br/>
                  <b>Created at: </b>{{ resource.created_at }}<br/>
                </div>
              </l-popup>
            </l-marker>
          </vue2-leaflet-markercluster>
      </l-map>
    </div>
    <div class="w-1/3 text-left p-5">
      <sui-tab>
        <sui-tab-pane title="Search Resources">
          <div>
            <sui-form>
              <sui-form-field>
                <label>Description</label>
                <input placeholder="Description" v-model="searchForm.description">
              </sui-form-field>
              <sui-form-field>
                <label>Capability</label>
                <sui-dropdown
                  fluid
                  :options="capabilities"
                  placeholder="Capability"
                  search
                  selection
                  v-model="searchForm.capability"
                />
              </sui-form-field>
              <div class="three fields">
                <sui-form-field>
                  <label>Latitude</label>
                  <input placeholder="Latitude" v-model="searchForm.lat">
                </sui-form-field>
                <sui-form-field>
                  <label>Longitude</label>
                  <input placeholder="Longitude" v-model="searchForm.lon">
                </sui-form-field>
                <sui-form-field>
                  <label>Radius</label>
                  <input placeholder="Radius" v-model="searchForm.radius">
                </sui-form-field>
              </div>
            </sui-form>
            <sui-message warning v-if="searched && resources.length===0">No results found for your search.</sui-message>
            <br/>
            <sui-button @click="search">Search</sui-button>
            <sui-button @click="clearFields">Clear Fields</sui-button>
          </div>
        </sui-tab-pane>
        <sui-tab-pane title="Search Context Data">
        </sui-tab-pane>
      </sui-tab>  
    </div>
  </div>
</template>

<script>
import { L, LMap, LTileLayer, LMarker, LPopup } from "vue2-leaflet";
import Vue2LeafletMarkercluster from "vue2-leaflet-markercluster";
import axios from "axios";

export default {
  name: "Example",
  components: {
    Vue2LeafletMarkercluster,
    LMap,
    LTileLayer,
    LMarker,
    LPopup
  },
  data() {
    return {
      searchForm: {
        description: "",
        lat: "",
        lon: "",
        radius: "",
        capability: ""
      },
      loading: false,
      searched: false,
      capabilities: [],
      resources: [],
      zoom: 10,
      center: L.latLng(-2.554965, -44.242386),
      url: "http://{s}.tile.osm.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      currentZoom: 10,
      currentCenter: L.latLng(-2.554965, -44.242386),
      showParagraph: false,
      mapOptions: {
        zoomSnap: 0.5
      }
    };
  },
  methods: {
    search() {
      this.searched = true;
      let query = "";
      query += this.searchForm.description
        ? "description=" + this.searchForm.description + "&"
        : "";
      query += this.searchForm.capability
        ? "capability=" + this.searchForm.capability + "&"
        : "";
      query += this.searchForm.lat ? "lat=" + this.searchForm.lat + "&" : "";
      query += this.searchForm.lon ? "lon=" + this.searchForm.lon + "&" : "";
      query += this.searchForm.radius
        ? "radius=" + this.searchForm.radius + "&"
        : "";
      if (query) {
        this.loading = "active";
        axios
          .get(
            "http://cidadesinteligentes.lsdi.ufma.br/catalog/resources/search?" +
              query
          )
          .then(response => response.data.resources)
          .then(resources => (this.resources = resources))
          .then(() => (this.loading = false));
      } else {
        this.loading = "active";
        axios
          .get("http://cidadesinteligentes.lsdi.ufma.br/catalog/resources")
          .then(response => response.data.resources)
          .then(resources => (this.resources = resources))
          .then(() => (this.loading = false));
      }
    },
    clearFields() {
      this.searchForm.description = "";
      this.searchForm.lat = "";
      this.searchForm.lon = "";
      this.searchForm.radius = "";
      this.searchForm.capability = "";
      this.searched = false;
    },
    getLocation(resource) {
      return L.latLng(resource.lat, resource.lon);
    },
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      this.currentCenter = center;
    }
  },
  mounted() {
    axios
      .get("http://cidadesinteligentes.lsdi.ufma.br/catalog/capabilities")
      .then(response => response.data.capabilities)
      .then(capabilities =>
        capabilities.map(cap => ({
          key: cap.name,
          value: cap.name,
          text: cap.name
        }))
      )
      .then(caps => (this.capabilities = caps));

    //{ key: "AL", value: "AL", text: "Alabama" }
  }
};
</script>