<template>
  <footer class="footer">
    <div>© PUSY 2022</div>
    <div><FooterPusyBeauty /></div>
    <a target="_blank" class="policy" href="https://pusy.beauty/privacy">
      Политика конфиденциальности
    </a>
  </footer>
</template>
<script setup>
import FooterPusyBeauty from "@/assets/footer-pusy-beauty.svg";
</script>
<style>
.footer {
  background-color: #e3e3df;
  display: none;
  grid-template-columns: 1fr;
  place-items: center;
  gap: 20px;
  padding: 12px;
  color: #8d8d8d;
  font-family: "RFTone-Bold", sans-serif;
}
.policy {
  color: #565656;
  font-weight: 600;
  font-family: "Montserrat", sans-serif;
}
@media (min-width: 1025px) {
  .footer {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
  }
}
</style>
