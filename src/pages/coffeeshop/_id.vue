<template>
  <div class="map_div">
    <div class="page-header page-header-xs settings-background" :style='`background-image: url("${place.preview_image}")`'>
      <div class="filter"></div>
    </div>
    <div class="container">
      <div class="row">
          <div class="col-md-12 ml-auto mr-auto text-center custom_title">
              <h2 class="place_title title-uppercase">{{ place.title }}</h2>
              <h3 class="title-uppercase"><b>{{ place.working_hours }}</b></h3>
              <h3 class="title-uppercase phone_str" v-if='place.registrations[0].phone'>
                <small>
                  <a class='place_link' :href="`tel:${place.registrations[0].phone}`">{{ place.registrations[0].phone }}</a>
                </small>
              </h3>
              <div class="row">
                <div class="ml-auto mr-auto text-center address_block">
                  <h5 class='title-uppercase text-left' v-for="tag, index in place.registrations">{{ tag.address }}</h5>
                </div>
              </div>

              <div class="col-md-12">
                <button v-on:click="socialRedirect(place.website)" v-if='place.website' class="btn btn-just-icon btn-link btn-twitter">
                    <i class="fa fa-safari" aria-hidden="true"></i>
                </button>
                <button v-on:click="socialRedirect(place.twitter)" v-if='place.twitter' class="btn btn-just-icon btn-link btn-twitter">
                    <i class="fa fa-twitter" aria-hidden="true"></i>
                </button>
                <button v-on:click="socialRedirect(place.facebook)" v-if='place.facebook' class="btn btn-just-icon btn-link btn-facebook">
                    <i class="fa fa-facebook" aria-hidden="true"></i>
                </button>
                <button v-on:click="socialRedirect(place.instagram)" v-if='place.instagram' class="btn btn-just-icon btn-link btn-instagram">
                    <i class="fa fa-instagram" aria-hidden="true"></i>
                </button>
							</div>
          </div>
      </div>
      <div class="custom_row" v-if='place.email'>
        <div class="col-md-6 ml-auto mr-auto text-center title custom_title email_title">
            <div class="title-uppercase text-center">
              <a class='place_link' :href="`mailto:${place.email}`">{{ place.email }}</a>
            </div>
        </div>
      </div>
      <div class="row" v-if='place.description'>
        <div class="col-md-6 ml-auto mr-auto text-center title custom_title">
          <div class="title-uppercase text-left">{{ place.description }}</div>
        </div>
        <div class="col-md-12 ml-auto mr-auto text-center title custom_title">
          <div class="col-md-12">
            <span v-for="tag, index in place.tags" class="label label-custom label_margin">#{{ tag }}</span>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-md-6 ml-auto mr-auto text-center title custom_title">
          <h3 class="option_title">Индекс кофейни</h3>
          <ul class='leaders text-left title-uppercase text-left'>
            <li>
              <span>Эспрессо</span>
              <span>{{ formatOption(place.espresso_price) }}</span>
            </li>
            <li>
              <span>Капучино</span>
              <span>{{ formatOption(place.cappuccino_price) }}</span>
            </li>
            <li>
              <span>Обжарка</span>
              <span>{{ formatOption(place.roasting) }}</span>
            </li>
            <li>
              <span>Продажа кофе в зёрнах</span>
              <span>{{ formatBoolean(place.sell_in_beans) }}</span>
            </li>
            <li>
              <span>Альтернатива</span>
              <span>{{ formatBoolean(place.alternate) }}</span>
            </li>
            <li>
              <span>Продажа аксессуаров</span>
              <span>{{ formatBoolean(place.merchandise) }}</span>
            </li>
            <li v-if='place.coffee_machine'>
              <span>Марка кофемашины</span>
              <span>{{ place.coffee_machine }}</span>
            </li>
            <li v-if='place.features'>
              <span>Дополнительно</span>
              <span>{{ place.features }}</span>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <div class="map-container-page">
      <gmap-map
      :center='place.registrations[0].location'
      :zoom='10'
      :options='{ styles: styles }'
      style='width: 100%; height: 300px'
      v-if='shouldRender'
      >
      <gmap-info-window
        v-if='infoWindowRegistration'
        :options='infoOptions'
        :position='infoWindowPos'
        :opened='infoWinOpen'
        @closeclick='infoWinOpen=false'
      >
      <div class="coffee_info">
        <h3 class="coffee_title">
          <router-link :to="{ name: 'coffeeshop-id', params: { id: place.slug }}">
            {{ place.title }}
          </router-link>
        </h3>
        <h5 class="price upcase">
          <strong>
            <a class='place_link phone_num' :href="`tel:${infoWindowRegistration.phone}`">{{ infoWindowRegistration.phone }}</a>
          </strong>
          </h5>
        <hr/>
        <p>{{ infoWindowRegistration.address }}</p>
        <br/>
        <span v-for="tag, index in place.tags" class="label label-custom label_margin">#{{ tag }}</span>
        </div>
      </gmap-info-window>

      <gmap-marker
        v-for='(m, index) in place.registrations'
        :position='m.location'
        :draggable='false'
        :clickable='true'
        icon='/map.png'
        @click='toggleInfoWindow(m, index)'
      ></gmap-marker>
    </gmap-map>
    </div>
    </div>
</template>
<script>
import apollo from '~/lib/apollo'
import { mapDefaults } from '~/lib/const'
import { customMapColors } from '~/lib/colors'
import gql from 'graphql-tag'
import google from 'vue2-google-maps'
import boolMixin from '~/helpers/view_helper.js'

export default {
  async asyncData ({ params, error }) {
    const { data } = await apollo.query({
      query: gql`{
        shop(id:"${params.id}") {
          id, title, description, slug, tags, registrations { phone, address,  location { lat, lng } }, working_hours, instagram, twitter, facebook, website, email, espresso_price, cappuccino_price, roasting, features, coffee_machine, sell_in_beans, alternate, merchandise, preview_image
        }
      }`,
      prefetch: true,
      fetchPolicy: 'network-only'
    })
    if (!data.shop) {
      error({'statusCode': 404, 'message': 'OK'})
    }
    return {
      ...mapDefaults,
      shouldRender: false,
      place: data.shop,
      mixins: [boolMixin],
      styles: customMapColors
    }
  },
  computed: {
    renderPanorama () {
      return this.infoWindowPos && this.infoWindowPos.lat && this.infoWindowPos.lng
    }
  },
  watch: {
    place (place) {
      if (place) {
        const bounds = new google.maps.LatLngBounds()
        this.$refs.map.$mapObject.fitBounds(bounds)
        this.$refs.map.$mapObject.setCenter(bounds.getCenter())
        this.$refs.map.$mapObject.event.addDomListener(window, 'resize', function () {
          this.$refs.map.$mapObject.setCenter(bounds.getCenter())
        })
      }
    }
  },
  methods: {
    updateCenter: function (center) {
      this.center = {
        lat: center.lat(),
        lng: center.lng()
      }
    },
    socialRedirect: function (link) {
      window.open(
        link,
        '_blank'
      )
    },
    toggleInfoWindow: function (marker, idx) {
      this.infoWindowPos = marker.location
      this.infoWindowRegistration = marker
      if (this.currentMidx === idx) {
        this.infoWinOpen = !this.infoWinOpen
      } else {
        this.infoWinOpen = true
        this.currentMidx = idx
      }
    }
  },
  mounted () {
    this.shouldRender = true
  }
}
</script>
