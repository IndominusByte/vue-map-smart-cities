<template>
  <div class="card">
    <img
      class="card-img-top img-fit"
      :src="value.icon"
      width="286"
      height="180"
    />
    <div class="card-body p-2">
      <h5 class="card-title text-truncate card-name">
        {{ value.name }}
      </h5>
      <div class="card-rating fs-12 mb-1">
        <span class="mr-1">{{ value.rating }}</span>
        <star-rating
          :rating="value.rating"
          :star-size="12"
          :read-only="true"
          :increment="0.01"
          :show-rating="false"
        ></star-rating>
        <span class="ml-1">({{ value.user_rating }})</span>
      </div>
      <p class="card-text text-truncate card-address">
        {{ value.address }}
      </p>
    </div>
    <!--/card-body-->
  </div>
  <!--/card-->
</template>

<script>
import StarRating from "vue-star-rating";

export default {
  props: ['value'],
  components:{
    StarRating
  }
}
</script>
