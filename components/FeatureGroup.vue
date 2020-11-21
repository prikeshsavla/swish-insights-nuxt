<template>
  <section class="features-icons bg-light text-center">
    <b-card
      no-body
      class="overflow-hidden"
      v-for="(feature, index) in features"
      :key="feature.title"
    >
      <b-row no-gutters :class="{ 'flex-row-reverse': index % 2 == 0 }">
        <b-col md="6">
          <b-card-img
            :src="feature.image"
            alt="Image"
            class="rounded-0"
          ></b-card-img>
        </b-col>
        <b-col md="6" align-self="center">
          <b-card-body :title="feature.title">
            <b-card-text>
              {{feature.summary}}
            </b-card-text>
          </b-card-body>
        </b-col>
      </b-row>
    </b-card>
  </section>
</template>

<script>
export default {
  data() {
    return {
      features: [
        {
          image: "img/bg-showcase-1.jpg",
          title: "Fully Responsive Design",
          summary:
            "This theme will look great on any device, no matter the size!"
        },
        {
          image: "img/bg-showcase-2.jpg",
          title: "Updated For Bootstrap 4",
          summary:
            "Newly improved, and full of great utility classes, Bootstrap 4 is leading the way in mobile responsive web development! All of the themes on Start Bootstrap are now using Bootstrap 4!"
        },
        {
          image: "img/bg-showcase-3.jpg",
          title: "Easy to Use & Customize",
          summary:
            "Landing Page is just HTML and CSS with a splash of SCSS for users who demand some deeper customization options. Out of the box, just add your content and images, and your new landing page will be ready to go!"
        }
      ]
    };
  }
};
</script>

<style lang="scss" scoped></style>
