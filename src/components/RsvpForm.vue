<template>
  <div class="wedding-page">

    <div class="wedding-container">
      <div class="header-content-inline">
          <h1 class="wedding-title">
            ♡ Darija  
            <span class="ampersand">&</span> 
            Ljubo ♡
          </h1>
          <p class="wedding-subtitle">Pozivamo vas da podijelite s nama ovaj poseban dan!</p>
        </div>
      <!-- Pozivnica kao slika -->
      <div class="invitation-section">
        <div class="invitation-card">
          <div class="invitation-image-wrapper">
            <img 
              src="/pozivnica.jpeg" 
              alt="Pozivnica za vjenčanje"
              class="invitation-image"
            />
          </div>
          
          <!-- Link za preuzimanje -->
          <a href="/pozivnica.jpeg" download="Darija&Ljubo.jpeg" class="download-link">
            <span>Preuzmi pozivnicu</span>
          </a>
        </div>
      </div>

      <!-- RSVP Forma -->
      <div class="rsvp-section">
        <div class="section-header">
          <h2 class="section-title">Potvrda dolaska</h2>
          <div class="divider"></div>
        </div>
        
        <div v-if="submitSuccess" class="success-card">
          <div class="success-animation">
            <div class="checkmark">✓</div>
          </div>
          <p>{{ successMessage }}</p>
        </div>

        <form v-else @submit.prevent="submitRsvp" class="rsvp-form">
          <!-- Osnovni podaci -->
          <div class="form-section">
            <h3 class="form-section-title">
            <span class="submit-icon">♡</span>
              Tvoji podaci
            </h3>
            
            <div class="form-row">
              <div class="form-group">
                <label for="ime">Ime </label>
                <input 
                  type="text" 
                  id="ime" 
                  v-model="form.ime" 
                  required
                  placeholder="Unesi svoje ime"
                  :class="{'error': errors.ime}"
                />
                <span v-if="errors.ime" class="error-text">{{ errors.ime[0] }}</span>
              </div>

              <div class="form-group">
                <label for="prezime">Prezime </label>
                <input 
                  type="text" 
                  id="prezime" 
                  v-model="form.prezime" 
                  required
                  placeholder="Unesi svoje prezime"
                  :class="{'error': errors.prezime}"
                />
                <span v-if="errors.prezime" class="error-text">{{ errors.prezime[0] }}</span>
              </div>
            </div>

          </div>

          <!-- Dolazak -->
          <div class="form-section">
            <h3 class="form-section-title">
              <span class="section-icon">♡</span>
              Potvrdi svoj dolazak
            </h3>
            
            <div class="radio-group">
              <div class="radio-cards">
                <label 
                  class="radio-card" 
                  :class="{'selected': form.dolazi === true}"
                >
                  <input 
                    type="radio" 
                    v-model="form.dolazi" 
                    :value="true"
                  />
                  <div class="radio-content">
                    <span class="radio-title">Dolazim</span>
                  </div>
                </label>

                <label 
                  class="radio-card" 
                  :class="{'selected': form.dolazi === false}"
                >
                  <input 
                    type="radio" 
                    v-model="form.dolazi" 
                    :value="false"
                  />
                  <div class="radio-content">
                    <span class="radio-title">Ne dolazim</span>
                  </div>
                </label>
              </div>
            </div>
          </div>

          <!-- Dodatni gosti -->
          <transition name="slide-fade">
            <div v-if="form.dolazi" class="form-section guests-section">
              <h3 class="form-section-title">
                Dodatni uzvanici
              </h3>
              
              <div class="form-group">
                <label for="broj_dodatnih">
                  Broj dodatnih osoba
                  <span class="help-text">(Maksimalno 5)</span>
                </label>
                <div class="number-input-wrapper">
                  <button 
                    type="button" 
                    class="number-btn"
                    @click="decrementGuests"
                    :disabled="form.broj_dodatnih <= 0"
                  >
                    −
                  </button>
                  <input 
                    type="number" 
                    id="broj_dodatnih" 
                    v-model.number="form.broj_dodatnih" 
                    min="0"
                    max="5"
                    @input="updateGuestsArray"
                    class="number-input"
                    readonly
                  />
                  <button 
                    type="button" 
                    class="number-btn"
                    @click="incrementGuests"
                    :disabled="form.broj_dodatnih >= 5"
                  >
                    +
                  </button>
                </div>
              </div>

              <!-- Lista gostiju -->
              <transition-group name="list" tag="div" class="guests-list">
                <div 
                  v-for="(guest, index) in form.dodatni_uzvanici" 
                  :key="index"
                  class="guest-card"
                >
                  <div class="guest-header">
                    <h4>
                      <span class="guest-number">{{ index + 1 }}</span>
                      Gost {{ index + 1 }}
                    </h4>
                    <button 
                      type="button" 
                      @click="removeGuest(index)"
                      class="remove-btn"
                      title="Ukloni gosta"
                    >
                      ×
                    </button>
                  </div>
                  <div class="guest-fields">
                    <div class="form-group">
                      <label>Ime </label>
                      <input 
                        type="text" 
                        v-model="guest.ime" 
                        required
                        placeholder="Ime gosta"
                      />
                    </div>
                    <div class="form-group">
                      <label>Prezime </label>
                      <input 
                        type="text" 
                        v-model="guest.prezime" 
                        required
                        placeholder="Prezime gosta"
                      />
                    </div>
                  </div>
                </div>
              </transition-group>
            </div>
          </transition>

          <!-- Submit -->
          <div class="form-actions">
            <button 
              type="submit" 
              class="submit-btn"
              :disabled="loading"
            >
              <span v-if="loading" class="loading-content">
                <span class="spinner"></span>
                <span>Šaljem potvrdu...</span>
              </span>
              <span v-else class="submit-content">
                <span class="submit-icon">♡</span>
                <span>Pošalji potvrdu</span>
                <span class="submit-icon">♡</span>
              </span>
            </button>
          </div>

          <div v-if="errorMessage" class="error-banner">
            <span class="error-icon">⚠️</span>
            <span>{{ errorMessage }}</span>
          </div>
        </form>
      </div>
      <div class="navigation-help">
        <h3 class="navigation-title">Trebate pomoć s dolaskom?</h3>

        <div class="navigation-links">
          <a 
            href="https://maps.app.goo.gl/UEg1d9o5k8oJvwGT9?g_st=aw"
            target="_blank"
            rel="noopener noreferrer"
            class="nav-link"
          >
            📍 Crkva Uznesenja BDM – Google Maps
          </a>

          <a 
            href="https://maps.app.goo.gl/rcRNH1gnrZWsiTVX9?g_st=aw"
            target="_blank"
            rel="noopener noreferrer"
            class="nav-link"
          >
            📍 Svadbeni salon Gospodar prstenova – Google Maps
          </a>
        </div>
      </div>
    </div>

    <!-- Footer -->
    <footer class="wedding-footer">
      <p>S ljubavlju ♡ Darija & Ljubo ♡</p>
    </footer>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'WeddingInvitation',
  
  data() {
    return {
      form: {
        ime: '',
        prezime: '',
        dolazi: null,
        broj_dodatnih: 0,
        dodatni_uzvanici: []
      },
      
      errors: {},
      loading: false,
      submitSuccess: false,
      successMessage: '',
      errorMessage: ''
    };
  },
  
  methods: {
    incrementGuests() {
      if (this.form.broj_dodatnih < 5) {
        this.form.broj_dodatnih++;
        this.updateGuestsArray();
      }
    },
    
    decrementGuests() {
      if (this.form.broj_dodatnih > 0) {
        this.form.broj_dodatnih--;
        this.updateGuestsArray();
      }
    },
    
    updateGuestsArray() {
      const count = parseInt(this.form.broj_dodatnih) || 0;
      const current = this.form.dodatni_uzvanici.length;
      
      if (count > current) {
        for (let i = current; i < count; i++) {
          this.form.dodatni_uzvanici.push({ ime: '', prezime: '' });
        }
      } else if (count < current) {
        this.form.dodatni_uzvanici = this.form.dodatni_uzvanici.slice(0, count);
      }
    },
    
    removeGuest(index) {
      this.form.dodatni_uzvanici.splice(index, 1);
      this.form.broj_dodatnih = this.form.dodatni_uzvanici.length;
    },
    
    async submitRsvp() {
      this.loading = true;
      this.errors = {};
      this.errorMessage = '';
      
      try {
        const response = await axios.post('http://localhost:8000/api/rsvps', this.form);
        
        this.submitSuccess = true;
        this.successMessage = response.data.message;
        
        window.scrollTo({ top: 0, behavior: 'smooth' });
        
        setTimeout(() => {
          this.resetForm();
        }, 10000);
        
      } catch (error) {
        console.error('Error:', error);
        
        if (error.response && error.response.status === 422) {
          this.errors = error.response.data.errors || {};
        } else {
          this.errorMessage = 'Došlo je do greške pri slanju. Molimo pokušajte ponovno.';
        }
        
        window.scrollTo({ top: document.querySelector('.rsvp-section').offsetTop - 100, behavior: 'smooth' });
      } finally {
        this.loading = false;
      }
    },
    
    resetForm() {
      this.form = {
        ime: '',
        prezime: '',
        dolazi: null,
        broj_dodatnih: 0,
        dodatni_uzvanici: []
      };
      this.submitSuccess = false;
      this.successMessage = '';
      this.errors = {};
    }
  }
};
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.wedding-page {
  min-height: 100vh;
  background: linear-gradient(0deg, #e3f2fd 0%, #bbdefb 50%, #e3f2fd 100%);
  font-family: 'Georgia', 'Times New Roman', serif;
}


@keyframes float {
  0%, 100% { transform: translateY(0px) rotate(0deg); }
  50% { transform: translateY(-10px) rotate(3deg); } /* Smanjeno sa -15px */
}

.header-content {
  position: relative;
  z-index: 1;
}
.header-content-inline {
  text-align: center;
  margin-bottom: 40px;
}

.header-content-inline .wedding-title {
  font-size: 2.5rem;
  font-family: 'Great Vibes', cursive;
  text-shadow: 2px 2px 6px rgba(0,0,0,0.15);
  color: black;
  margin-bottom: 8px;
}

.header-content-inline .ampersand {
  font-family: 'Times New Roman', Times, serif;
  font-style: italic;
  font-size: 0.8em;
  color: black;
}

.header-content-inline .wedding-subtitle {
  font-size: 1rem;
  font-style: italic;
  color: black;
  font-weight: 300;
  margin: 0;
}

.wedding-title {
  font-size: 2.5rem; /* Smanjeno sa 3rem */
  color: black;
  margin: 0 0 8px 0; /* Smanjeno sa 10px */
  font-family: 'Great Vibes', cursive;
  font-weight: 400;
  text-shadow: 2px 2px 6px rgba(0,0,0,0.15);
  animation: fadeIn 1s ease-in;
}

.ampersand {
  font-family: 'Times New Roman', Times, serif;
  font-style: italic;
  color: black;
  font-size: 0.8em; /* da se malo bolje uklopi */
}

.wedding-subtitle {
  font-size: 1rem; /* Smanjeno sa 1.2rem */
  color: black;
  margin: 0;
  font-style: italic;
  font-weight: 300;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-20px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Container */
.wedding-container {
  max-width: 950px;
  margin: 0 auto;
  padding: 40px 20px;
}

/* Invitation Section */
.invitation-section {
  margin-bottom: 50px;
}

.invitation-card {
  background: white;
  padding: 25px;
  border-radius: 20px;
  box-shadow: 0 8px 30px rgba(33, 150, 243, 0.15);
  text-align: center;
}

.invitation-image-wrapper {
  position: relative;
  width: 100%;
  max-width: 500px;
  margin: 0 auto 25px;
  border-radius: 15px;
  overflow: hidden;
  box-shadow: 0 6px 25px rgba(33, 150, 243, 0.2);
  border: 4px solid #c5e2eb;
  background: #f5f5f5;
}

.invitation-image {
  width: 100%;
  height: auto;
  display: block;
  transition: transform 0.3s ease;
}

.invitation-image-wrapper:hover .invitation-image {
  transform: scale(1.02);
}

.download-link {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 12px 28px;
  background: linear-gradient(135deg, #acd3f2, #acd3f2);
  color: white;
  text-decoration: none;
  border-radius: 50px;
  font-size: 16px;
  font-weight: 600;
  transition: all 0.3s;
  box-shadow: 0 4px 15px rgba(33, 150, 243, 0.3);
}

.download-link:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 25px rgba(33, 150, 243, 0.4);
  background: linear-gradient(135deg, #acd3f2, #acd3f2);
}


/* Section Styling */
.section-header {
  text-align: center;
  margin-bottom: 35px;
}

.section-title {
  font-size: 2rem;
  color: #1976d2;
  margin: 0 0 12px 0;
  font-family: 'Great Vibes', cursive;
  font-weight: 400;
}

.divider {
  width: 80px;
  height: 3px;
  background: linear-gradient(90deg, transparent, #64b5f6, transparent);
  margin: 0 auto;
}



/* RSVP Section */
.rsvp-section {
  background: white;
  padding: 45px 35px;
  border-radius: 20px;
  box-shadow: 0 8px 30px rgba(33, 150, 243, 0.15);
}

.rsvp-form {
  max-width: 650px;
  margin: 0 auto;
}

/* Success Card */
.success-card {
  text-align: center;
  padding: 50px 35px;
  background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
  border-radius: 20px;
  box-shadow: 0 6px 25px rgba(33, 150, 243, 0.15); /* Plavi tonovi */
}

.success-animation {
  margin-bottom: 20px;
}

.checkmark {
  display: inline-block;
  width: 70px;
  height: 70px;
  border-radius: 50%;
  background: #42a5f5; /* plava umjesto zelena */
  color: white;
  font-size: 45px;
  line-height: 70px;
  animation: scaleIn 0.5s ease-out;
  box-shadow: 0 4px 20px rgba(33, 150, 243, 0.3); /* plavi ton */
}

@keyframes scaleIn {
  0% { transform: scale(0); opacity: 0; }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); opacity: 1; }
}

.success-card h3 {
  font-size: 1.8rem;
  color: #1976d2;
  margin: 0 0 12px 0;
  font-family: 'Great Vibes', cursive;
  font-weight: 400;
}

.success-card p {
  font-size: 1.1rem;
  color: #1976d2; /* tamno plava */
  margin: 0;
}

/* Form Sections */
.form-section {
  background: #f8fbff;
  padding: 28px;
  border-radius: 15px;
  margin-bottom: 25px;
  border: 2px solid #e3f2fd;
}

.form-section-title {
  font-size: 1.4rem;
  color: #1976d2;
  margin: 0 0 22px 0;
  font-family: 'Georgia', serif;
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: 300;
}

.section-icon {
  font-size: 22px;
}

.guests-section {
  background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
  border-color: #90caf9;
}

/* Form Elements */
.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 18px;
}

.form-group {
  margin-bottom: 18px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  color: #1565c0;
  font-weight: 300;
  font-size: 14px;
}

.help-text {
  font-size: 12px;
  color: #666;
  font-weight: normal;
  margin-left: 5px;
}

.form-group input[type="text"]
{
  width: 100%;
  padding: 13px 16px;
  border: 2px solid #bbdefb;
  border-radius: 10px;
  font-size: 15px;
  font-family: inherit;
  transition: all 0.3s;
  background: white;
}

.form-group input:focus {
  outline: none;
  border-color: #42a5f5;
  box-shadow: 0 0 0 3px rgba(66, 165, 245, 0.1);
}

.form-group input.error {
  border-color: #e74c3c;
  background: #fff5f5;
}

.error-text {
  color: #e74c3c;
  font-size: 13px;
  margin-top: 5px;
  display: flex;
  align-items: center;
  gap: 5px;
}

.error-text::before {
  content: '⚠️';
}

/* Radio Cards */
.radio-group {
  margin: 0;
}

.main-label {
  display: block;
  margin-bottom: 18px;
  color: #1565c0;
  font-weight: 300;
  font-size: 15px;
}

.radio-cards {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 18px;
}

.radio-card {
  position: relative;
  display: block;
  padding: 22px;
  background: white;
  border: 3px solid #bbdefb;
  border-radius: 15px;
  cursor: pointer;
  transition: all 0.3s;
}

.radio-card input[type="radio"] {
  position: absolute;
  opacity: 0;
}

.radio-card:hover {
  border-color: #64b5f6;
  transform: translateY(-2px);
  box-shadow: 0 5px 18px rgba(33, 150, 243, 0.15);
}

.radio-card.selected {
  border-color: #42a5f5;
  background: linear-gradient(135deg, #e3f2fd, #bbdefb);
  box-shadow: 0 5px 20px rgba(33, 150, 243, 0.25);
}

.radio-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  gap: 8px;
}

.radio-emoji {
  font-size: 35px;
  margin-bottom: 3px;
}

.radio-title {
  font-size: 16px;
  font-weight: 600;
  color: #1565c0;
}


@keyframes checkPop {
  0% { transform: scale(0); }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); }
}

/* Number Input */
.number-input-wrapper {
  display: flex;
  align-items: center;
  gap: 0;
  max-width: 170px;
}

.number-btn {
  width: 42px;
  height: 42px;
  border: 2px solid #64b5f6;
  background: white;
  color: #1976d2;
  font-size: 22px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.number-btn:first-child {
  border-radius: 10px 0 0 10px;
}

.number-btn:last-child {
  border-radius: 0 10px 10px 0;
}

.number-btn:hover:not(:disabled) {
  background: #64b5f6;
  color: white;
}

.number-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.number-input {
  width: 86px;
  height: 42px;
  border: 2px solid #64b5f6;
  border-left: none;
  border-right: none;
  text-align: center;
  font-size: 19px;
  font-weight: bold;
  color: #1976d2;
  background: white;
}

/* Guests List */
.guests-list {
  margin-top: 22px;
}

.guest-card {
  background: white;
  padding: 22px;
  border-radius: 12px;
  margin-bottom: 18px;
  border: 2px solid #bbdefb;
  box-shadow: 0 2px 8px rgba(33, 150, 243, 0.08);
  transition: all 0.3s;
}

.guest-card:hover {
  box-shadow: 0 4px 15px rgba(33, 150, 243, 0.15);
}

.guest-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 18px;
}

.guest-header h4 {
  margin: 0;
  color: #1976d2;
  font-size: 16px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.guest-number {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  background: #64b5f6;
  color: white;
  border-radius: 50%;
  font-size: 14px;
  font-weight: bold;
}

.remove-btn {
  width: 32px;
  height: 32px;
  border: none;
  background: #e74c3c;
  color: white;
  border-radius: 50%;
  font-size: 22px;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
  line-height: 1;
}

.remove-btn:hover {
  background: #c0392b;
  transform: rotate(90deg);
}

.guest-fields {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
}

/* Submit Button */
.form-actions {
  margin-top: 35px;
}

.submit-btn {
  width: 100%;
  padding: 18px;
  background: linear-gradient(135deg, #82bbe8, #82bbe8);
  color: white;
  border: none;
  border-radius: 12px;
  font-size: 18px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 5px 18px rgba(33, 150, 243, 0.35);
}

.submit-btn:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(33, 150, 243, 0.4);
  background: linear-gradient(135deg, #acd3f2, #9acdf5);
}

.submit-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
}

.submit-content,
.loading-content {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
}

.submit-icon {
  font-size: 22px;
}

.spinner {
  width: 20px;
  height: 20px;
  border: 3px solid rgba(255,255,255,0.3);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Error Banner */
.error-banner {
  margin-top: 22px;
  padding: 16px 22px;
  background: linear-gradient(135deg, #ffebee, #ffcdd2);
  color: #c62828;
  border-radius: 12px;
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: 600;
  border: 2px solid #ffcdd2;
  box-shadow: 0 4px 15px rgba(244, 67, 54, 0.15);
}

.error-icon {
  font-size: 20px;
}

/* Footer */
.wedding-footer {
  text-align: center;
  padding: 25px 20px; /* Smanjeno sa 35px */
  background: #ffff; /*linear-gradient(135deg, #acd3f2 0%, #42a5f5 100%);*/
  color: #acd3f2;
  font-size: 17px;
  box-shadow: 0 -4px 20px rgba(0,0,0,0.1);
}

.wedding-footer p {
  margin: 0;
  font-family: 'Great Vibes', cursive;
  color: black;
  font-weight: 400;
  font-size: 20px; /* Smanjeno sa 22px */
}

/* Animations */
.slide-fade-enter-active {
  transition: all 0.4s ease-out;
}

.slide-fade-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter-from {
  transform: translateY(-25px);
  opacity: 0;
}

.slide-fade-leave-to {
  transform: translateY(-20px);
  opacity: 0;
}

.list-enter-active,
.list-leave-active {
  transition: all 0.4s ease;
}

.list-enter-from {
  opacity: 0;
  transform: translateX(-40px);
}

.list-leave-to {
  opacity: 0;
  transform: translateX(40px);
}

.list-move {
  transition: transform 0.4s ease;
}

/* Responsive */
@media (max-width: 768px) {
  .wedding-title {
    font-size: 2rem; /* Smanjeno sa 2.3rem */
  }
  
  .wedding-subtitle {
    font-size: 0.95rem;
  }
  
  .section-title {
    font-size: 1.7rem;
  }
  
  .invitation-image-wrapper {
    max-width: 100%;
  }
  
  .rsvp-section {
    padding: 35px 25px;
  }
  
  .form-row,
  .radio-cards,
  .guest-fields {
    grid-template-columns: 1fr;
  }
  
  .form-section {
    padding: 22px 18px;
  }
  
  .header-section {
    padding: 25px 20px; /* Smanjeno sa 40px */
  }
}

@media (max-width: 480px) {
  .wedding-container {
    padding: 30px 15px;
  }
  
  .invitation-card {
    padding: 20px 15px;
  }
  
  .rsvp-section {
    padding: 28px 18px;
  }
}

/* Navigation Help Section */
.navigation-help {
  margin-top: 10px;
  padding: 30px 20px;
  text-align: center;
}

.navigation-title {
  font-family: 'Times New Roman';
  font-size: 1.4rem;
  color: #1976d2;
  margin-bottom: 10px;
  font-weight: 400;
}

.navigation-text {
  font-size: 0.95rem;
  color: #1565c0;
  margin-bottom: 20px;
}

.navigation-links {
  display: flex;
  justify-content: center;
  gap: 20px;
  flex-wrap: wrap;
}

.nav-link {
  text-decoration: none;
  color: #1976d2;
  font-weight: 600;
  font-size: 0.95rem;
  padding: 8px 14px;
  border-radius: 20px;
  transition: all 0.3s ease;
}

.nav-link:hover {
  background: rgba(255,255,255,0.4);
  transform: translateY(-2px);
}

</style>