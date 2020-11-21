<template>
  <div>
    <b-navbar toggleable="sm" type="light" variant="light" class="fixed-bottom-sm">
      <div class="container">
        <b-navbar-toggle target="nav-text-collapse"></b-navbar-toggle>

        <b-navbar-brand>Swish</b-navbar-brand>

        <b-collapse id="nav-text-collapse" is-nav>
          <b-navbar-nav class="ml-auto">
            <b-nav-text class="mr-3">Leaderboards</b-nav-text>
            <b-button variant="primary" class="my-2 my-sm-0" type="submit"
              >Search</b-button
            >
          </b-navbar-nav>
        </b-collapse>
      </div>
    </b-navbar>
  </div>
</template>

<script>
export default {};
</script>

<style lang="scss" scoped>
.fixed-bottom-sm {
  @media (max-width: 767px) {
        position: fixed;
    right: 0;
    bottom: 0;
    left: 0;
    z-index: 1030;
  }
}
</style>
