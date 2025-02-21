<script setup>

  import { ref, onMounted } from "vue";
  import axios from "axios";

  const personnes = ref([]);
  const projets = ref([]);
  const personneCourante = ref(null);
  const projetCourant = ref(null);
  const role = ref("");
  const pourcentage = ref(10);
  const errorMessage = ref("");

  async function chargementDonnees() {
    try {
      const responsePersonnes = await axios.get("/api/personnes");
      console.log("Personnes :", responsePersonnes.data);
      personnes.value = responsePersonnes.data._embedded ? responsePersonnes.data._embedded.personnes : [];

      const responseProjets = await axios.get("/api/projets");
      console.log("Projets :", responseProjets.data);
      projets.value = responseProjets.data._embedded ? responseProjets.data._embedded.projets : [];
    } catch (error) {
      console.error("Erreur de récupération des données :", error);
    }
  }

  const ajoutParticipation = async () => {
    try {
      console.log(personneCourante.value)
      console.log(projetCourant.value)
      console.log(role.value)
      await axios.post(
        "http://localhost:8989/api/gestion/participation",
        {},
        {
          params: {
            matricule: personneCourante.value,
            codeProjet: projetCourant.value,
            role: role.value,
            pourcentage: pourcentage.value / 100,
          },
          headers: {
            'Content-Type': 'application/json',
          },
        }
      );
      alert("Participation enregistrée avec succès !");
    } catch (error) {
      if (error.response) {
        errorMessage.value = error.response.data.message
      } else {
        errorMessage.value = 'Erreur lors de l’enregistrement.'
      }
    }
  };

  onMounted(async () => {
    await chargementDonnees();
  });

</script>

<template>
  <main class="enregistrerParticipation">
    <section>
      <h2>Enregistrer une participation</h2>
    </section>
    <form @submit.prevent="ajoutParticipation">
      <section>
        <label for="personne">Personne :</label>
        <select id="personne" v-model="personneCourante" required>
          <option v-for="personne in personnes" :key="personne.matricule" :value="personne.matricule">
            {{ personne.prenom }} {{ personne.nom }} - {{ personne.poste }}
          </option>
        </select>
      </section>

      <section class="form-group">
        <label for="projet">Projet :</label>
        <select id="projet" v-model="projetCourant" required>
          <option v-for="projet in projets" :key="projet.id" :value="projet.id">
            {{ projet.nom }} ({{ projet.debut }})
          </option>
        </select>
      </section>

      <div class="form-group">
        <label for="role">Rôle :</label>
        <input type="text" id="role" v-model="role" required />
      </div>

      <div class="form-group">
        <label for="pourcentage">Pourcentage :</label>
        <input type="range" id="pourcentage" v-model="pourcentage" min="0" max="100" step="1" required />
        <span>{{ pourcentage }}%</span>
      </div>

      <button type="submit">Enregistrer</button>
    </form>

    <div v-if="errorMessage" class="error">{{ errorMessage }}</div>

  </main>
</template>

<style scoped>

  .enregistrerParticipation {
    margin: 2rem auto;
    max-width: 600px;
    background-color: #f9f9f9;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
  }

  .form-group {
    margin-bottom: 15px;
  }

  label {
    display: block;
    margin-bottom: 5px;
  }

  select, input {
    width: 100%;
    padding: 8px;
    box-sizing: border-box;
  }

  button {
    padding: 10px 15px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }

  button:hover {
    background-color: #0056b3;
  }

  .error {
    color: red;
    margin-top: 10px;
  }

</style>
