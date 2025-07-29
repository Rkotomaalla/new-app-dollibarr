<template>
  <div class="p-6 max-w-lg mx-auto bg-white shadow-lg rounded-xl">
    <h2 class="text-xl font-bold mb-4">Réinitialiser Dolibarr</h2>

    <p class="mb-2">Sélectionne les éléments à effacer :</p>
    <div class="mb-4">
      <label><input type="checkbox" v-model="modules.products"> Produits</label><br>
      <label><input type="checkbox" v-model="modules.warehouses"> Entrepôts</label><br>
      <label><input type="checkbox" v-model="modules.stocks"> Stocks</label><br>
      <label><input type="checkbox" v-model="modules.fabrications"> Fabrications</label><br>
      <label><input type="checkbox" v-model="modules.orders"> Ordres de fabrication</label>
    </div>

    <button @click="resetAll" class="bg-red-500 text-white px-4 py-2 rounded hover:bg-red-600">
      Réinitialiser
    </button>

    <div v-if="loading" class="mt-3 text-blue-500">⏳ Suppression en cours...</div>
    <div v-if="success" class="mt-3 text-green-600">✅ Données effacées avec succès !</div>
    <div v-if="error" class="mt-3 text-red-600">❌ Erreur : {{ errorMessage }}</div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "ResetDolibarrAll",
  data() {
    return {
      modules: {
        products: false,
        warehouses: false,
        stocks: false,
        fabrications: false,
        orders: false
      },
      loading: false,
      success: false,
      error: false,
      errorMessage: "",
    };
  },
  methods: {
    async resetAll() {

      this.loading = true;
      this.success = false;
      this.error = false;

      const token = "H715aQG9dLjE0G1BUknwyyh17nP0iR4N"; 
      const baseUrl = "http://localhost/api/index.php";
      const headers = { DOLAPIKEY: token };

      try {
        // 1️⃣ Supprimer les produits
        if (this.modules.products) {
          const res = await axios.get(`${baseUrl}/products`, { headers });
          for (let p of res.data) {
            await axios.delete(`${baseUrl}/products/${p.id}`, { headers });
          }
        }

        // 2️⃣ Supprimer les entrepôts
        if (this.modules.warehouses) {
          const res = await axios.get(`${baseUrl}/warehouses`, { headers });
          console.log(res.data);
          for (let w of res.data) {
            await axios.delete(`${baseUrl}/warehouses/${w.id}`, { headers });
          }
        }

        // 3️⃣ Supprimer les mouvements de stock
        if (this.modules.stocks) {
          const res = await axios.get(`${baseUrl}/stockmovements`, { headers });
          console.log(res.data);
          for (let s of res.data) {
            await axios.delete(`${baseUrl}/stockmovements/${s.id}`, { headers });
          }
        }

        // 4️⃣ Supprimer les fabrications (MO)
        if (this.modules.fabrications || this.modules.orders) {
          const res = await axios.get(`${baseUrl}/mrp_mo`, { headers });
          console.log(res.data);
          for (let m of res.data) {
            await axios.delete(`${baseUrl}/mrp_mo/${m.id}`, { headers });
          }
        }

        this.success = true;
      } catch (err) {
        this.error = true;
        this.errorMessage = err.message;
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>
