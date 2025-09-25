<template>
    <div id="app-vue-container" class="main-container container mt-4" :class="{ 'is-premium': isPremium }">

        <!-- Header -->
        <header class="d-flex justify-content-between align-items-center mb-4">
            <div class="d-flex align-items-center">
                <div
                    style="width: 44px; height: 44px; border-radius: 14px; background: linear-gradient(45deg, #00c6fb 0%, #005bea 100%); display: flex; align-items: center; justify-content: center; box-shadow: 0 4px 15px -5px rgba(0, 198, 251, 0.4);">
                    <i class="material-icons" style="color: #fff; font-size: 30px; vertical-align: middle;">spa</i>
                </div>
                <h1 class="h3 ms-3 mb-0">AI Lifestyle Manager</h1>
            </div>
            <button @click="openProfile" class="btn btn-outline-primary" style="width:44px;height:44px;display:flex;align-items:center;justify-content:center;border-radius:50%;">
                <i style="margin-bottom: 4px;" class="material-icons">person</i>
            </button>
        </header>

        <main>
            <!-- Today's Overview -->
            <section class="mb-5">
                <h2 class="h5 mb-3 text-muted">Riepilogo di Oggi</h2>
                <div class="row">
                   <div class="col-md-6 mb-4">
                        <div @click="openMealsModal" class="card h-100" style="cursor: pointer;">
                            <div class="card-body d-flex align-items-start">
                                <i class="material-icons text-primary me-3 mt-1" style="font-size: 2.5rem;">restaurant</i>
                                <div>
                                    <h6 class="card-title">{{ nextMeal.name }}</h6>
                                    <p class="card-text small text-muted">{{ nextMeal.desc }}</p>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="col-md-6 mb-4">
                        <div @click="openWorkoutModal" class="card h-100" style="cursor: pointer;">
                            <div class="card-body d-flex align-items-start">
                                <i class="material-icons text-primary me-3 mt-1" style="font-size: 2.5rem;">fitness_center</i>
                                <div>
                                    <h6 class="card-title">Allenamento di Oggi</h6>
                                    <p class="card-text small text-muted" v-if="plan[todayStr]">{{ plan[todayStr].workout.title }}</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            
            <!-- Full Plan -->
            <section>
                <h2 class="h5 mb-3 text-muted">Il Tuo Piano Settimanale</h2>
                
                <ul class="nav nav-tabs nav-justified mb-3" id="pills-tab" role="tablist">
                    <li class="nav-item" role="presentation">
                        <a class="nav-link" :class="{ active: activeTab === 'meal' }" @click="activeTab = 'meal'" type="button" role="tab">
                            <i class="material-icons me-2">restaurant_menu</i>Piano Alimentare
                        </a>
                    </li>
                    <li class="nav-item" role="presentation">
                        <a class="nav-link" :class="{ active: activeTab === 'workout' }" @click="activeTab = 'workout'" type="button" role="tab">
                             <i class="material-icons me-2">fitness_center</i>Piano Esercizi
                        </a>
                    </li>
                </ul>

                    <div class="tab-content" id="pills-tabContent">
                        <div class="tab-pane fade" :class="{ 'show active': activeTab === 'meal' }" role="tabpanel">
                            <div class="row row-cols-1 row-cols-md-2 g-4">
                                <div v-for="(dayData, dayName) in plan" :key="`meal-${dayName}`" class="col">
                                    <div class="card h-100" :class="{ 'is-today': dayName === todayStr }">
                                        <div class="card-body">
                                            <h5 class="card-title text-primary">{{ dayName }}</h5>
                                            <div class="mt-3 small">
                                                <div v-for="(meal, index) in dayData.meal" :key="index" class="mb-2">
                                                    <strong>{{ meal.name }}<span v-if="meal.time"> ({{ meal.time }})</span>:</strong>
                                                    {{ meal.description || meal.desc }}
                                                </div>
                                            </div>
                                        </div>
                                        <div class="card-footer bg-transparent border-0 text-end">
                                            <button @click="openEditMealModal(dayName)" class="btn btn-outline-primary btn-square" style="width:45px;height:45px;display:inline-flex;align-items:center;justify-content:center;">
                                                <i class="material-icons">edit</i>
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    <div class="tab-pane fade" :class="{ 'show active': activeTab === 'workout' }" role="tabpanel">
                        <div class="row row-cols-1 row-cols-md-2 g-4">
                             <div v-for="(dayData, dayName) in plan" :key="`workout-${dayName}`" class="col">
                                <div class="card h-100" :class="{ 'is-today': dayName === todayStr }">
                                    <div class="card-body">
                                        <h5 class="card-title text-primary">{{ dayName }}</h5>
                                        <div class="mt-3 small">
                                             <h6 class="card-subtitle mb-2 text-muted">{{ dayData.workout.title }}</h6>
                                             <p class="card-text">{{ dayData.workout.details.map(e => e.name).join(', ') }}</p>
                                        </div>
                                    </div>
                                    <div class="card-footer bg-transparent border-0 text-end">
                                        <button @click="openEditWorkoutModal(dayName)" class="btn btn-outline-primary btn-square" style="width:45px;height:45px;display:inline-flex;align-items:center;justify-content:center;">
                                            <i class="material-icons">edit</i>
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
        </main>

        <!-- Modals -->
        <div class="modal fade" id="editMealModal" ref="editMealModalRef" tabindex="-1">
            <div class="modal-dialog modal-lg modal-dialog-scrollable">
            <div class="modal-content">
                <div class="modal-header">
                <h5 class="modal-title">Modifica Piano Alimentare per <span class="text-primary">{{ currentEditingDay }}</span></h5>
                <button @click="closeModal" type="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                <button @click="handleSuggestMeals" class="btn btn-primary btn-sm mb-4 w-100 d-flex align-items-center justify-content-center">
                    <i class="material-icons me-2">auto_awesome</i> ✨ Genera Idee con l'IA
                </button>
                <div v-if="editableMeals.length > 0">
                    <div v-for="(item, index) in editableMeals" :key="index" class="d-flex mb-3 meal-edit-card position-relative">
                    <!-- Controlli ordine -->
                    <div class="d-flex flex-column me-2 align-self-center">
                        <button @click="moveMealUp(index)" :disabled="index === 0" class="btn btn-outline-secondary btn-sm mb-1" style="width:32px;height:32px;display:flex;align-items:center;justify-content:center;">
                            <i class="material-icons" style="font-size:16px;">keyboard_arrow_up</i>
                        </button>
                        <button @click="moveMealDown(index)" :disabled="index === editableMeals.length - 1" class="btn btn-outline-secondary btn-sm" style="width:32px;height:32px;display:flex;align-items:center;justify-content:center;">
                            <i class="material-icons" style="font-size:16px;">keyboard_arrow_down</i>
                        </button>
                    </div>
                    <div class="flex-grow-1">
                        <button
                            @click="removeMeal(index)"
                            class="btn btn-link text-danger p-0 meal-remove-btn"
                            style="position: absolute; top: 8px; right: 8px; z-index: 10;"
                        >
                            <i class="material-icons">delete</i>
                        </button>
                        <div class="row g-2">
                        <div class="col-7">
                            <div class="form-outline flex-fill mb-3">
                            <input type="text" v-model="item.name" class="form-control" :id="`meal-name-${index}`" />
                            <label class="form-label" :for="`meal-name-${index}`" :class="{ filled: item.name }">Nome Pasto (es. Colazione)</label>
                            </div>
                        </div>
                        <div class="col-5">
                            <div class="form-outline flex-fill mb-3">
                            <input type="time" v-model="item.time" class="form-control" :id="`meal-time-${index}`" />
                            <label class="form-label" :for="`meal-time-${index}`" :class="{ filled: item.time }">Orario</label>
                            </div>
                        </div>
                        </div>
                        <div class="form-outline flex-fill me-2">
                        <textarea v-model="item.desc" class="form-control" :id="`meal-desc-${index}`" rows="3"></textarea>
                        <label class="form-label" :for="`meal-desc-${index}`" :class="{ filled: item.desc }">Descrizione</label>
                        </div>
                    </div>
                    </div>
                </div>
                <button @click="addMeal" class="btn btn-sm btn-outline-primary mb-4 d-flex align-items-center">
                    <i class="material-icons me-1">add</i>
                    <span>Aggiungi Pasto</span>
                </button>
                </div>
                <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal" @click="closeModal">Annulla</button>
                <button type="button" class="btn btn-primary" @click="handleSaveMealChanges">Salva Modifiche</button>
                </div>
            </div>
            </div>
        </div>

        <div class="modal fade" id="editWorkoutModal" ref="editWorkoutModalRef" tabindex="-1">
            <div class="modal-dialog modal-lg modal-dialog-scrollable">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Modifica Allenamento per <span class="text-primary">{{ currentEditingDay }}</span></h5>
                        <button  @click="closeModal" type="button" class="btn-close" data-bs-dismiss="modal"></button>
                    </div>
                    <div class="modal-body">
                        <div class="form-outline mb-3" v-if="editableWorkout">
                            <input type="text" v-model="editableWorkout.title" class="form-control" id="workout-title-edit"/>
                             <label class="form-label" for="workout-title-edit" :class="{ filled: editableWorkout.title }">Titolo Allenamento</label>
                        </div>
                        <button @click="handleGenerateWorkout" class="btn btn-primary btn-sm mb-4 w-100 d-flex align-items-center justify-content-center">
                            <i class="material-icons me-2">auto_awesome</i> ✨ Genera Esercizi con l'IA
                        </button>
                        <div v-if="editableWorkout">
                            <div v-for="(exercise, index) in editableWorkout.details" :key="index" class=" exercise-edit-card mb-3 position-relative">
                                <div class="d-flex">
                                    <!-- Controlli ordine -->
                                    <div class="d-flex flex-column me-2 align-self-center">
                                        <button @click="moveExerciseUp(index)" :disabled="index === 0" class="btn btn-outline-secondary btn-sm mb-1" style="width:32px;height:32px;display:flex;align-items:center;justify-content:center;">
                                            <i class="material-icons" style="font-size:16px;">keyboard_arrow_up</i>
                                        </button>
                                        <button @click="moveExerciseDown(index)" :disabled="index === editableWorkout.details.length - 1" class="btn btn-outline-secondary btn-sm" style="width:32px;height:32px;display:flex;align-items:center;justify-content:center;">
                                            <i class="material-icons" style="font-size:16px;">keyboard_arrow_down</i>
                                        </button>
                                    </div>
                                    <div class="flex-grow-1">
                                        <button
                                            @click="removeExercise(index)"
                                            class="btn btn-link text-danger p-0 meal-remove-btn"
                                            style="position: absolute; top: 8px; right: 8px; z-index: 10;"
                                        >
                                            <i class="material-icons">delete</i>
                                        </button>
                                        <div class="form-outline mb-3">
                                            <input type="text" v-model="exercise.name" class="form-control" :id="`ex-name-${index}`" />
                                            <label class="form-label" :for="`ex-name-${index}`" :class="{ filled: exercise.name }">Nome Esercizio</label>
                                        </div>
                                        <div class="input-group  mb-3">
                                            <div class="form-outline flex-fill">
                                                <input type="url" v-model="exercise.link" class="form-control" :id="`ex-link-${index}`" :value="exercise.link ?? ''"/>
                                                <label class="form-label" :for="`ex-link-${index}`" :class="{ filled: exercise.link }">Link YouTube (opzionale)</label>
                                            </div>
                                            <button @click="searchYoutube(exercise.name)" class="btn btn-outline-secondary" type="button">Cerca</button>
                                        </div>
                                        <div class="form-outline">
                                            <textarea v-model="exercise.desc" class="form-control" :id="`ex-desc-${index}`" rows="2"></textarea>
                                            <label class="form-label" :for="`ex-desc-${index}`" :class="{ filled: exercise.desc }">Descrizione Esercizio</label>
                                        </div>
                                     
                                    </div>
                                </div>
                            </div>
                        </div>
                        <button @click="addExercise" class="btn btn-sm btn-outline-primary mb-4 d-flex align-items-center">
                            <i class="material-icons me-1">add</i>
                            <span>Aggiungi Esercizio</span>
                        </button>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal"  @click="closeModal">Annulla</button>
                        <button type="button" class="btn btn-primary" @click="handleSaveWorkoutChanges">Salva Modifiche</button>
                    </div>
                </div>
            </div>
        </div>

        
        <div class="modal fade" id="workoutModal" ref="workoutModalRef" tabindex="-1">
          <div class="modal-dialog modal-lg modal-dialog-scrollable">
            <div class="modal-content">
              <div class="modal-header">
                 <button @click="navigateWorkoutDay(-1)" class="btn btn-link px-2"><i class="material-icons">chevron_left</i></button>
                <h5 class="modal-title text-center mx-auto">Dettaglio Allenamento</h5>
                <button @click="navigateWorkoutDay(1)" class="btn btn-link px-2"><i class="material-icons">chevron_right</i></button>
              </div>
              <div class="modal-body">
                <h3 class="text-primary mb-4 text-center">{{ workoutModalContent.title }}</h3>
            
                <div class="row row-cols-1 g-4 mt-2">
                    <div v-if="!workoutModalContent.details || workoutModalContent.details.length === 0" class="col">
                        <p class="text-muted col-12">Nessun esercizio specifico trovato.</p>
                    </div>
                    <div v-else v-for="(exercise, index) in workoutModalContent.details" :key="index" class="col">
                         <div class="card exercise-card" :class="{ 'exercise-completed': isExerciseCompleted(exercise.name) }" @click="handleExerciseClick(exercise.name)">
                            <div class="card-body d-flex align-items-start">
                                <i class="material-icons text-primary me-3 mt-1" style="font-size: 2.5rem;">fitness_center</i>
                                <div class="flex-grow-1">
                                    <h6 class="card-title mb-1">
                                        <i class="material-icons completion-icon" style="color: var(--completed-color); vertical-align: middle; font-size: 1.2em;">check_circle</i>
                                        {{ exercise.name }}
                                    </h6>
                                    <div class="card-text" v-if="exercise.link">
                                        <div v-if="isPremium" class="ratio ratio-16x9 mt-2">
                                            <iframe :src="getYoutubeEmbedUrl(exercise.link)" :title="`Video for ${exercise.name}`" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
                                        </div>
                                        <div v-else class="alert alert-warning mt-2 small locked-feature" @click.stop="premiumModalInstance?.show()">
                                            Video disponibile con Premium <i class="material-icons premium-lock-icon">lock</i>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
              </div>
              <div class="modal-footer">
                <button @click="closeModal" type="button" class="btn btn-secondary" data-bs-dismiss="modal">Chiudi</button>
              </div>
            </div>
          </div>    
        </div>


        <div class="modal fade" id="mealsModal" ref="mealsModalRef" tabindex="-1">
            <div class="modal-dialog modal-lg modal-dialog-scrollable">
            <div class="modal-content">
                <div class="modal-header">
                <button @click="navigateMealDay(-1)" class="btn btn-link px-2"><i class="material-icons">chevron_left</i></button>
                <h5 class="modal-title text-center mx-auto">Pasti del {{ currentViewingMealDay }}</h5>
                <button @click="navigateMealDay(1)" class="btn btn-link px-2"><i class="material-icons">chevron_right</i></button>
                </div>
                <div class="modal-body">
                <div class="row row-cols-1 g-3">
                    <div v-if="mealsModalContent.length === 0" class="col">
                    <p class="text-muted">Nessun pasto programmato.</p>
                    </div>
                    <div v-else v-for="(meal, index) in mealsModalContent" :key="index" class="col">
                    <div class="card meal-detail-card">
                        <div class="card-body d-flex align-items-center">
                        <i class="material-icons text-primary me-3" style="font-size: 2.5rem;">{{ meal.icon }}</i>
                        <div>
                            <h6 class="card-title mb-1">{{ plan[currentViewingMealDay]?.meal[index]?.name }}<span v-if="plan[currentViewingMealDay]?.meal[index]?.time"> ({{ plan[currentViewingMealDay]?.meal[index]?.time }})</span></h6>
                            <p class="card-text small text-muted mb-0">{{ plan[currentViewingMealDay]?.meal[index]?.description || plan[currentViewingMealDay]?.meal[index]?.desc }}</p>
                        </div>
                        </div>
                    </div>
                    </div>
                </div>
                </div>
                <div class="modal-footer">
                <button @click="closeModal" type="button" class="btn btn-secondary" data-bs-dismiss="modal">Chiudi</button>
                </div>
            </div>
            </div>
        </div>
        <div class="modal fade" id="loadingModal" ref="loadingModalRef" tabindex="-1" aria-hidden="true" data-mdb-backdrop="static" data-mdb-keyboard="false">
          <div class="modal-dialog modal-dialog-centered">
            <div class="modal-content border-0" style="background-color: transparent; box-shadow: none;">
              <div class="modal-body text-center">
                <div class="spinner-border text-primary" role="status" style="width: 3rem; height: 3rem;">
                  <span class="visually-hidden">Loading...</span>
                </div>
                <p class="mt-3 text-white">Elaborazione in corso...</p>
              </div>
            </div>
          </div>
        </div>

        <!-- AI Suggestions Modal -->
        <div class="modal fade" id="aiSuggestionsModal" ref="aiSuggestionsModalRef" tabindex="-1" aria-hidden="true">
          <div class="modal-dialog modal-lg modal-dialog-scrollable">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title">✨ Suggerimenti dall'IA</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
              </div>
              <div class="modal-body">
                  <div class="list-group list-group-flush ">
                       <a href="#" v-for="(suggestion, index) in aiSuggestions" :key="index" @click.prevent="addSuggestedMeal(suggestion)" class="list-group-item list-group-item-action suggestion-item text-white">
                           <div class="d-flex w-100 justify-content-between">
                                <h6 class="mb-1">{{ suggestion.name }}</h6>
                              </div>
                              <p class="mb-1 small">{{ suggestion.desc }}</p>
                       </a>
                  </div>
              </div>
              <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Chiudi</button>
              </div>
            </div>
          </div>
        </div>
        
        <!-- Premium Modal -->
                <div class="modal fade" id="premiumModal" ref="premiumModalRef" tabindex="-1" aria-hidden="true">
                    <div class="modal-dialog modal-dialog-centered">
                        <div class="modal-content">
                            <div class="modal-header border-0">
                                <h5 class="modal-title">
                                        <i class="material-icons me-2" style="color: var(--premium-color);">workspace_premium</i>
                                        Passa a Premium
                                </h5>
                                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                            </div>
                            <div class="modal-body">
                                <p>Sblocca tutte le funzionalità per un'esperienza completa!</p>
                                <ul class="list-group list-group-flush">
                                        <li class="list-group-item bg-transparent"><i class="material-icons me-2 text-primary">check</i> Utilizzo illimitato delle funzioni IA</li>
                                        <li class="list-group-item bg-transparent"><i class="material-icons me-2 text-primary">check</i> Video tutorial degli esercizi integrati</li>
                                </ul>
                            </div>
                            <div class="modal-footer border-0">
                                <button type="button" class="btn btn-premium w-100" @click="unlockPremium">Sblocca Ora</button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Modal: Seleziona tipo pasto per AI -->
                <div class="modal fade" id="aiMealTypeModal" ref="aiMealTypeModalRef" tabindex="-1" aria-hidden="true">
                    <div class="modal-dialog modal-dialog-centered">
                        <div class="modal-content">
                            <div class="modal-header">
                                <h5 class="modal-title">Seleziona tipo di pasto</h5>
                                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"  @click="closeModal"></button>
                            </div>
                            <div class="modal-body">
                                <div class="form-outline flex-fill me-2 mb-3">
                                    <input type="text" v-model="aiMealTypeInput" class="form-control" id="ai-meal-type-input" />
                                    <label class="form-label" for="ai-meal-type-input" :class="{ filled: aiMealTypeInput }">Tipo Pasto (es. Colazione proteica, Pranzo leggero)</label>
                                </div>
                            </div>
                            <div class="modal-footer">
                                <button type="button" class="btn btn-secondary" @click="closeModal">Annulla</button>
                                <button type="button" class="btn btn-primary" @click="confirmAiMealType">Conferma</button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Modal: Profilo Utente -->
                <div class="modal fade" id="profileModal" ref="profileModalRef" tabindex="-1" aria-hidden="true">
                    <div class="modal-dialog modal-lg modal-dialog-scrollable">
                        <div class="modal-content">
                            <div class="modal-header">
                                <h5 class="modal-title">
                                    <i class="material-icons me-2 text-primary">person</i>
                                    Il Tuo Profilo
                                </h5>
                                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close" @click="closeModal"></button>
                            </div>
                            <div class="modal-body">
                                <div class="row g-3">
                                    <!-- Sesso -->
                                    <div class="col-12">
                                        <div class="form-outline">
                                            <select v-model="userProfile.gender" class="form-control" id="profile-gender">
                                                <option value="M">Maschio</option>
                                                <option value="F">Femmina</option>
                                            </select>
                                            <label class="form-label" for="profile-gender" :class="{ filled: userProfile.gender }">Sesso</label>
                                        </div>
                                    </div>
                                    
                                    <!-- Data di nascita -->
                                    <div class="col-12">
                                        <div class="form-outline">
                                            <input type="date" v-model="userProfile.birthDate" class="form-control" id="profile-birth-date" />
                                            <label class="form-label" for="profile-birth-date" :class="{ filled: userProfile.birthDate }">Data di Nascita</label>
                                        </div>
                                    </div>
                                    
                                    <!-- Altezza e Peso attuale -->
                                    <div class="col-6">
                                        <div class="form-outline">
                                            <input type="number" v-model="userProfile.height" class="form-control" id="profile-height" min="100" max="250" step="1" />
                                            <label class="form-label" for="profile-height" :class="{ filled: userProfile.height }">Altezza (cm)</label>
                                        </div>
                                    </div>
                                    
                                    <div class="col-6">
                                        <div class="form-outline">
                                            <input type="number" v-model="userProfile.currentWeight" class="form-control" id="profile-current-weight" min="30" max="300" step="0.1" />
                                            <label class="form-label" for="profile-current-weight" :class="{ filled: userProfile.currentWeight }">Peso Attuale (kg)</label>
                                        </div>
                                    </div>
                                    
                                    <!-- Peso obiettivo -->
                                    <div class="col-12">
                                        <div class="form-outline">
                                            <input type="number" v-model="userProfile.targetWeight" class="form-control" id="profile-target-weight" min="30" max="300" step="0.1" />
                                            <label class="form-label" for="profile-target-weight" :class="{ filled: userProfile.targetWeight }">Peso Obiettivo (kg)</label>
                                        </div>
                                    </div>
                                    
                                    <!-- Livello di Attività -->
                                    <div class="col-12">
                                        <div class="form-outline">
                                            <select v-model="userProfile.activityLevel" class="form-control" id="profile-activity-level">
                                                <option value="nessuna">Nessuna attività fisica</option>
                                                <option value="poca">Poca attività (1-2 volte/settimana)</option>
                                                <option value="media">Mediamente attivo (3-4 volte/settimana)</option>
                                                <option value="alta">Molto attivo (5+ volte/settimana)</option>
                                            </select>
                                            <label class="form-label" for="profile-activity-level" :class="{ filled: userProfile.activityLevel }">Livello di Attività Fisica</label>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="modal-footer">
                                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal" @click="closeModal">Annulla</button>
                                <button type="button" class="btn btn-primary" @click="saveUserProfile">Salva Profilo</button>
                            </div>
                        </div>
                    </div>
                </div>


        
    </div>
</template>

<script setup>
import { onMounted, reactive, ref, computed, nextTick } from 'vue';

const mdb = window.mdb;

// AI Meal Type Modal State
const aiMealTypeInput = ref('');
const aiMealTypeModalRef = ref(null);


// --- STATE ---
const plan = reactive({});
const isPremium = ref(false);
const currentEditingDay = ref(null);
const editableMeals = ref([]);
const editableWorkout = ref(null);
const activeTab = ref('meal');
const currentViewingWorkoutDay = ref(null);
const workoutModalContent = reactive({ title: '', details: [] });
const workoutCompletionStatus = reactive({});
const currentViewingMealDay = ref(null);
const mealsModalContent = reactive([]);
const aiSuggestions = ref([]);

// User Profile State
const userProfile = reactive({
    gender: '',
    birthDate: '',
    height: '',
    currentWeight: '',
    targetWeight: '',
    activityLevel: ''
});


// Modal Refs
const editMealModalRef = ref(null);
let mdbEditMealModal = null;
const editWorkoutModalRef = ref(null);
let mdbEditWorkoutModal = null;
const workoutModalRef = ref(null);
const profileModalRef = ref(null);
let mdbProfileModal = null;
let mdbWorkoutModal = null;
const mealsModalRef = ref(null);
let mdbMealsModal = null;
const loadingModalRef = ref(null);
let mdbLoadingModal = null;
const aiSuggestionsModalRef = ref(null);
let mdbAiSuggestionsModal = null;
const premiumModalRef = ref(null);
let premiumModalInstance = null;

// Timer Refs
const timerProgressRingRef = ref(null);
const timerDisplayRef = ref(null);
const timerToggleBtnRef = ref(null);
const timerState = reactive({
    interval: null,
    totalSeconds: 0,
    remainingSeconds: 0,
    isPaused: true,
    circumference: 0,
});
const defaultPlan = {
    "Lunedì": {
        meal: [
            { name: "Colazione", time: "8:00", description: "200g yogurt greco 0% + 30g avena + 10g noci" },
            { name: "Spuntino", time: "10:30", description: "1 mela" },
            { name: "Pranzo", time: "13:00", description: "150g riso integrale + 150g pollo + verdure + 10g olio EVO" },
            { name: "Spuntino pomeriggio", time: "16:30", description: "15g mandorle" },
            { name: "Cena", time: "19:30", description: "200g pesce bianco + verdure + 15g olio EVO" }
        ],
        workout: {
            title: "Forza – Petto & Tricipiti",
            details: [
                { name: "Panca piana manubri 4x8–10", link: "" },
                { name: "Croci panca 3x12", link: "" },
                { name: "Dip 3xmax", link: "" },
                { name: "Push-up 3x15–20", link: "" },
                { name: "Plank 3x45s", link: "" }
            ]
        }
    },
    "Martedì": {
        meal: [
            { name: "Colazione", time: "8:00", description: "2 uova strapazzate + 1 fetta pane integrale" },
            { name: "Spuntino", time: "10:30", description: "1 yogurt 0%" },
            { name: "Pranzo", time: "13:00", description: "130g pasta integrale + 150g tonno al naturale + verdure + 10g olio EVO" },
            { name: "Spuntino pomeriggio", time: "16:30", description: "2 gallette con 10g burro arachidi" },
            { name: "Cena", time: "19:30", description: "180g salmone + verdure + 15g olio EVO" }
        ],
        workout: {
            title: "Cardio – Corsa o camminata veloce",
            details: [
                { name: "30–40 min corsa media o 45–50 min camminata veloce", link: "" },
                { name: "Stretching gambe", link: "" }
            ]
        }
    },
    "Mercoledì": {
        meal: [
            { name: "Colazione", time: "8:00", description: "200ml latte scremato + 30g muesli" },
            { name: "Spuntino", time: "10:30", description: "1 banana piccola" },
            { name: "Pranzo", time: "13:00", description: "200g patate dolci + 150g tacchino + verdure + 15g olio EVO" },
            { name: "Spuntino pomeriggio", time: "16:30", description: "15g pistacchi" },
            { name: "Cena", time: "19:30", description: "180g pollo alla piastra + verdure" }
        ],
        workout: {
            title: "Forza – Schiena & Bicipiti",
            details: [
                { name: "Rematore manubri 4x8–10", link: "" },
                { name: "Trazioni 3x10 o lat pulldown con elastici", link: "" },
                { name: "Curl manubri 3x12–15", link: "" },
                { name: "Pull-over 3x12", link: "" },
                { name: "Crunch 3x20", link: "" }
            ]
        }
    },
    "Giovedì": {
        meal: [
            { name: "Colazione", time: "8:00", description: "200g yogurt greco 0% + 30g fiocchi d’avena + 10g semi di chia" },
            { name: "Spuntino", time: "10:30", description: "1 pera" },
            { name: "Pranzo", time: "13:00", description: "150g riso basmati + 150g merluzzo + verdure + 10g olio EVO" },
            { name: "Spuntino pomeriggio", time: "16:30", description: "1 barretta proteica leggera (100 kcal)" },
            { name: "Cena", time: "19:30", description: "200g manzo magro + verdure" }
        ],
        workout: {
            title: "Cardio – Bici o corsa leggera",
            details: [
                { name: "40–60 min bici ritmo medio", link: "" },
                { name: "Se piove: corsa 25 min + stretching", link: "" }
            ]
        }
    },
    "Venerdì": {
        meal: [
            { name: "Colazione", time: "8:00", description: "2 uova sode + 1 fetta pane integrale" },
            { name: "Spuntino", time: "10:30", description: "1 yogurt 0%" },
            { name: "Pranzo", time: "13:00", description: "130g pasta integrale + 150g sgombro + verdure + 10g olio EVO" },
            { name: "Spuntino pomeriggio", time: "16:30", description: "15g noci" },
            { name: "Cena", time: "19:30", description: "180g pollo o tacchino + verdure + 15g olio EVO (pasto libero sera se vuoi)" }
        ],
        workout: {
            title: "Forza – Gambe & Spalle",
            details: [
                { name: "Squat manubri 4x12", link: "" },
                { name: "Affondi 3x12 per gamba", link: "" },
                { name: "calf raise", link: "" },
                { name: "Shoulder press 4x10", link: "" },
                { name: "Alzate laterali 3x12–15", link: "" },
                { name: "Plank laterale 3x30s", link: "" }
            ]
        }
    },
    "Sabato": {
        meal: [
            { name: "Colazione", time: "8:00", description: "200g yogurt greco 0% + 30g muesli" },
            { name: "Spuntino", time: "10:30", description: "1 frutto di stagione" },
            { name: "Pranzo", time: "13:00", description: "200g patate + 150g pollo + verdure + 10g olio EVO" },
            { name: "Spuntino pomeriggio", time: "16:30", description: "2 gallette con 10g crema di arachidi" },
            { name: "Cena", time: "19:30", description: "200g salmone o pesce spada + verdure" }
        ],
        workout: {
            title: "Cardio lungo – Bici o corsa lenta",
            details: [
                { name: "60–90 min bici o 50–60 min corsa lenta", link: "" },
                { name: "10 min addome", link: "" }
            ]
        }
    },
    "Domenica": {
        meal: [
            { name: "Colazione", time: "8:00", description: "200ml latte scremato + 30g avena + 10g mandorle" },
            { name: "Spuntino", time: "10:30", description: "1 mela o 1 yogurt" },
            { name: "Pranzo", time: "13:00", description: "150g riso integrale + 150g tacchino + verdure + 10g olio EVO" },
            { name: "Spuntino pomeriggio", time: "16:30", description: "15g nocciole" },
            { name: "Cena", time: "19:30", description: "200g pesce bianco + verdure + 15g olio EVO" }
        ],
        workout: {
            title: "Recupero attivo – camminata, stretching",
            details: [
                { name: "Camminata 45–60 min", link: "" },
                { name: "Stretching completo o yoga leggero", link: "" }
            ]
        }
    }
};

// --- COMPUTED ---
const todayStr = computed(() => {
    const days = ["Domenica", "Lunedì", "Martedì", "Mercoledì", "Giovedì", "Venerdì", "Sabato"];
    return days[new Date().getDay()];
});

const nextMeal = computed(() => {
    const todayMeals = plan[todayStr.value]?.meal;
    if (!todayMeals || todayMeals.length === 0) {
        return { name: "Nessun piano per oggi", desc: "" };
    }
    const now = new Date();
    const currentTime = now.getHours() * 60 + now.getMinutes();

    // Parse meals with time
    const mealsWithTime = todayMeals
        .map(meal => {
            if (!meal.time) return null;
            console.log("meal", meal);
            const [h, m] = meal.time.split(':').map(Number);
            if (isNaN(h) || isNaN(m)) return null;
            const time = h * 60 + m;
            return { ...meal, time };
        })
        .filter(Boolean)
        .sort((a, b) => a.time - b.time);

    const upcomingMeal = mealsWithTime.find(meal => meal.time > currentTime);

    if (upcomingMeal) {
        return { name: `Prossimo Pasto: ${upcomingMeal.name}`, desc: upcomingMeal.description || upcomingMeal.desc || "" };
    }
    return { name: "Pasti di oggi completati!", desc: "Buon riposo!" };
});

// --- METHODS ---
function loadPlan() {
    const savedPlan = localStorage.getItem('wellnessPlan');
    Object.assign(plan, savedPlan ? JSON.parse(savedPlan) : defaultPlan);
}

function savePlan() {
    localStorage.setItem('wellnessPlan', JSON.stringify(plan));
}
async function openEditMealModal(day) {
    currentEditingDay.value = day;
    // Now plan[day].meal is an array of objects: { name, time, description }
    editableMeals.value = plan[day].meal.map(meal => ({
        name: meal.name || '',
        time: meal.time || '',
        desc: meal.description || meal.desc || ''
    }));
    mdbEditMealModal.show();
    await nextTick();
    document.querySelectorAll('#editMealModal .form-outline').forEach(el => new mdb.Input(el).init());
}

function handleSaveMealChanges() {
    // Save as array of objects: { name, time, description }
    plan[currentEditingDay.value].meal = editableMeals.value.map(item => ({
        name: item.name || '',
        time: item.time || '',
        description: item.desc || ''
    }));
    savePlan();
    mdbEditMealModal.hide();
}

function closeModal() {
    mdbEditMealModal.hide();
    mdbEditWorkoutModal.hide();
    mdbWorkoutModal.hide();
    mdbMealsModal.hide();
    mdbAiSuggestionsModal.hide();
    if (mdbProfileModal) mdbProfileModal.hide();
}

function addMeal() {
    editableMeals.value.push({ name: null, desc: null });
}

function removeMeal(index) {
    editableMeals.value.splice(index, 1);
}

function moveMealUp(index) {
    if (index > 0) {
        const item = editableMeals.value.splice(index, 1)[0];
        editableMeals.value.splice(index - 1, 0, item);
    }
}

function moveMealDown(index) {
    if (index < editableMeals.value.length - 1) {
        const item = editableMeals.value.splice(index, 1)[0];
        editableMeals.value.splice(index + 1, 0, item);
    }
}

async function openEditWorkoutModal(day) {
    currentEditingDay.value = day;
    editableWorkout.value = JSON.parse(JSON.stringify(plan[day].workout));
    mdbEditWorkoutModal.show();
     await nextTick();
    document.querySelectorAll('#editWorkoutModal .form-outline').forEach(el => new mdb.Input(el).init());
}

function handleSaveWorkoutChanges() {
    plan[currentEditingDay.value].workout = editableWorkout.value;
    savePlan();
    mdbEditWorkoutModal.hide();
}

function addExercise() {
    if (editableWorkout.value) {
        editableWorkout.value.details.push({ name: '', link: '' });
    }
}

function removeExercise(index) {
     if (editableWorkout.value) {
        editableWorkout.value.details.splice(index, 1);
    }
}

function moveExerciseUp(index) {
    if (editableWorkout.value && index > 0) {
        const item = editableWorkout.value.details.splice(index, 1)[0];
        editableWorkout.value.details.splice(index - 1, 0, item);
    }
}

function moveExerciseDown(index) {
    if (editableWorkout.value && index < editableWorkout.value.details.length - 1) {
        const item = editableWorkout.value.details.splice(index, 1)[0];
        editableWorkout.value.details.splice(index + 1, 0, item);
    }
}

function searchYoutube(exerciseName) {
    const query = exerciseName.replace(/\d+x\d+.*/, '').trim();
    if (query) {
        window.open(`https://www.youtube.com/results?search_query=esecuzione+${encodeURIComponent(query)}`, '_blank');
    }
}

function isExerciseCompleted(exerciseName) {
    const day = currentViewingWorkoutDay.value;
    return workoutCompletionStatus[day]?.includes(exerciseName);
}

function handleExerciseClick(exerciseName) {
    const day = currentViewingWorkoutDay.value;
    if (!day) return;

    if (!workoutCompletionStatus[day]) {
        workoutCompletionStatus[day] = [];
    }
    const completedList = workoutCompletionStatus[day];
    const index = completedList.indexOf(exerciseName);

    if (index > -1) {
        completedList.splice(index, 1);
    } else {
        completedList.push(exerciseName);
    }
    localStorage.setItem('workoutCompletionStatus', JSON.stringify(workoutCompletionStatus));
}

function renderWorkoutModalContent(day) {
    currentViewingWorkoutDay.value = day;
    const workout = plan[day]?.workout;
    if (workout) {
        workoutModalContent.title = `${day}: ${workout.title}`;
        workoutModalContent.details = workout.details;
    } else {
        workoutModalContent.title = `${day}: Nessun Allenamento`;
        workoutModalContent.details = [];
    }
}

function openWorkoutModal() {
    const days = ["Domenica", "Lunedì", "Martedì", "Mercoledì", "Giovedì", "Venerdì", "Sabato"];
    const today = days[new Date().getDay()];
    renderWorkoutModalContent(today);
    mdbWorkoutModal.show();
}

function navigateWorkoutDay(direction) {
    const days = ["Lunedì", "Martedì", "Mercoledì", "Giovedì", "Venerdì", "Sabato", "Domenica"];
    const currentIndex = days.indexOf(currentViewingWorkoutDay.value);
    const newIndex = (currentIndex + direction + 7) % 7;
    renderWorkoutModalContent(days[newIndex]);
}

function getYoutubeEmbedUrl(url) {
    if (!url) return '';
    const regex = /(?:https?:\/\/)?(?:www\.)?(?:youtube\.com\/(?:[^\/\n\s]+\/\S+\/|(?:v|e(?:mbed)?)\/|\S*?[?&]v=)|youtu\.be\/)([a-zA-Z0-9_-]{11})/;
    const match = url.match(regex);
    return match ? `https://www.youtube.com/embed/${match[1]}` : '';
}

// Meal Detail Modal Logic
function renderMealsModalContent(day) {
    currentViewingMealDay.value = day;
    const mealPlan = plan[day];
    mealsModalContent.splice(0); // Clear array

    if (mealPlan && mealPlan.meal) {
        mealPlan.meal.forEach((item) => {
            console.log("item", item);
            mealsModalContent.push({
                name: item.name,
                desc: item.description || item.desc || '',
                icon: getMealIcon(item.name),
                time: item.time || ''
            });
        });
    }
}

function openMealsModal() {
    const days = ["Domenica", "Lunedì", "Martedì", "Mercoledì", "Giovedì", "Venerdì", "Sabato"];
    const today = days[new Date().getDay()];
    renderMealsModalContent(today);
    mdbMealsModal.show();
}

function navigateMealDay(direction) {
    const days = ["Lunedì", "Martedì", "Mercoledì", "Giovedì", "Venerdì", "Sabato", "Domenica"];
    const currentIndex = days.indexOf(currentViewingMealDay.value);
    const newIndex = (currentIndex + direction + 7) % 7;
    renderMealsModalContent(days[newIndex]);
}

function getMealIcon(mealName) {
    const name = mealName.toLowerCase();
    if (name.includes('colazione')) return 'free_breakfast';
    if (name.includes('pranzo')) return 'lunch_dining';
    if (name.includes('cena')) return 'dinner_dining';
    if (name.includes('spuntino')) return 'bakery_dining';
    return 'restaurant';
}



function checkAndUpdateAiUsage() {
    if (isPremium.value) return true;
    const today = new Date().toISOString().split('T')[0];
    const usageData = JSON.parse(localStorage.getItem('aiUsage') || '{}');
    const currentCount = usageData[today] || 0;
    
    if (currentCount >= 14) {
        alert('Hai raggiunto il limite di 14 interazioni con l\'IA per oggi. Passa a Premium per un uso illimitato!');
        premiumModalInstance?.show();
        return false;
    }
    
    usageData[today] = currentCount + 1;
    localStorage.setItem('aiUsage', JSON.stringify(usageData));
    return true;
}

async function callGeminiApi(prompt, schema = null) {

    // In your script, import the key:
    const apiKey = import.meta.env.VITE_GEMINI_API_KEY;
    const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
    const payload = { contents: [{ parts: [{ text: prompt }] }] };
    if (schema) {
        payload.generationConfig = { responseMimeType: "application/json", responseSchema: schema };
    }
    try {
        const response = await fetch(apiUrl, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload)
        });
        if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
        const result = await response.json();
        if (result.candidates?.[0]?.content?.parts?.[0]?.text) {
            return result.candidates[0].content.parts[0].text;
        } else {
            throw new Error("Invalid API response");
        }
    } catch (error) {
        console.error("Gemini API call failed:", error);
        return null;
    }
}

async function handleGenerateWorkout() {
    if (!checkAndUpdateAiUsage()) return;

    const workoutTitle = editableWorkout.value.title.trim();
    if (!workoutTitle) {
        alert("Per favore, inserisci un titolo per l'allenamento (es. Gambe, Petto, Cardio).");
        return;
    }
    
    mdbLoadingModal.show();
    
    // Costruisci il profilo utente per il prompt
    const profileContext = buildUserProfileContext();
    const geminiPrompt = `Sei un esperto personal trainer. ${profileContext}
    
    Crea un piano di allenamento con 4 esercizi per "${workoutTitle}". Considera il profilo dell'utente per personalizzare intensità e tipologia degli esercizi.
    
    Fornisci solo una risposta in formato JSON, seguendo questo schema: \`{"details": [{"name": "[Nome Esercizio 1 (serie x reps)]", "link": ""}]}\`. Non aggiungere testo o markdown.`;
    const workoutSchema = { type: "OBJECT", properties: { details: { type: "ARRAY", items: { type: "OBJECT", properties: { name: { type: "STRING" }, link: { type: "STRING" } }, required: ['name', 'link'] } } }, required: ['details'] };
    
    const responseText = await callGeminiApi(geminiPrompt, workoutSchema);
    mdbLoadingModal.hide();

    if (responseText) {
        try {
            const generatedPlan = JSON.parse(responseText);
            if (generatedPlan.details) {
                editableWorkout.value.details = generatedPlan.details;
            }
        } catch (e) {
            alert("C'è stato un errore nel generare l'allenamento. Riprova.");
        }
    } else {
        alert("Non è stato possibile contattare l'IA. Controlla la tua connessione e riprova.");
    }
}

async function handleSuggestMeals() {
    if (!checkAndUpdateAiUsage()) return;
    aiMealTypeInput.value = '';
    if (aiMealTypeModalRef.value) {
        new mdb.Modal(aiMealTypeModalRef.value).show();
    }
}

async function confirmAiMealType() {
    const mealType = aiMealTypeInput.value.trim();
    if (!mealType) return;
    if (aiMealTypeModalRef.value) {
        mdb.Modal.getInstance(aiMealTypeModalRef.value)?.hide();
    }


    mdbLoadingModal.show();
    
    // Costruisci il profilo utente per il prompt
    const profileContext = buildUserProfileContext();
    const geminiPrompt = `Sei un esperto nutrizionista. ${profileContext}
    
    Suggerisci 3 idee per "${mealType}". Considera il profilo dell'utente per personalizzare le porzioni e i macronutrienti in base al suo obiettivo di peso.
    
    Fornisci solo una risposta in formato JSON, seguendo questo schema: \`[{"name": "[Nome Pasto 1]", "desc": "[Descrizione]"}]\`. Il nome del pasto deve includere un orario suggerito. Non aggiungere testo o markdown.`;
    const mealSchema = { type: "ARRAY", items: { type: "OBJECT", properties: { name: { type: "STRING" }, desc: { type: "STRING" } }, required: ['name', 'desc'] } };
    const responseText = await callGeminiApi(geminiPrompt, mealSchema);
    mdbLoadingModal.hide();
    if (responseText) {
        try {
            aiSuggestions.value = JSON.parse(responseText);
            if (aiSuggestions.value.length > 0) {
                mdbAiSuggestionsModal.show();
            }
        } catch(e) {
            alert("C'è stato un errore nel generare i suggerimenti. Riprova.");
        }
    } else {
        alert("Non è stato possibile contattare l'IA. Controlla la tua connessione e riprova.");
    }
}

function buildUserProfileContext() {
    const profile = userProfile;
    let context = "Profilo utente: ";
    
    if (profile.gender) {
        context += `Sesso: ${profile.gender === 'M' ? 'Maschio' : 'Femmina'}. `;
    }
    
    if (profile.birthDate) {
        const age = new Date().getFullYear() - new Date(profile.birthDate).getFullYear();
        context += `Età: ${age} anni. `;
    }
    
    if (profile.height) {
        context += `Altezza: ${profile.height} cm. `;
    }
    
    if (profile.currentWeight) {
        context += `Peso attuale: ${profile.currentWeight} kg. `;
    }
    
    if (profile.targetWeight) {
        const weightDiff = profile.currentWeight - profile.targetWeight;
        if (weightDiff > 0) {
            context += `Obiettivo: perdere ${weightDiff.toFixed(1)} kg (peso target: ${profile.targetWeight} kg). `;
        } else if (weightDiff < 0) {
            context += `Obiettivo: aumentare di ${Math.abs(weightDiff).toFixed(1)} kg (peso target: ${profile.targetWeight} kg). `;
        } else {
            context += `Obiettivo: mantenere il peso attuale. `;
        }
    }
    
    if (profile.activityLevel) {
        const activityLabels = {
            'nessuna': 'Nessuna attività fisica',
            'poca': 'Poca attività (1-2 volte/settimana)', 
            'media': 'Mediamente attivo (3-4 volte/settimana)',
            'alta': 'Molto attivo (5+ volte/settimana)'
        };
        context += `Livello di attività: ${activityLabels[profile.activityLevel]}. `;
    }
    
    return context.trim() || "Profilo utente non disponibile.";
}

function addSuggestedMeal(suggestion) {
    editableMeals.value.push({ ...suggestion });
    mdbAiSuggestionsModal.hide();
}

function openProfile() {
    loadUserProfile();
    mdbProfileModal.show();
}

function loadUserProfile() {
    const savedProfile = localStorage.getItem('userProfile');
    if (savedProfile) {
        Object.assign(userProfile, JSON.parse(savedProfile));
    }
}

function saveUserProfile() {
    localStorage.setItem('userProfile', JSON.stringify(userProfile));
    mdbProfileModal.hide();
}

function checkPremiumStatus() {
    if (localStorage.getItem('wellnessPremium') === 'true') {
        isPremium.value = true;
        document.body.classList.add('is-premium');
    }
}

function unlockPremium() {
    isPremium.value = true;
    localStorage.setItem('wellnessPremium', 'true');
    document.body.classList.add('is-premium');
    premiumModalInstance.hide();
}



// --- LIFECYCLE HOOKS ---
onMounted(() => {
    isPremium.value = localStorage.getItem('wellnessPremium') === 'true';
    if (isPremium.value) {
        document.body.classList.add('is-premium');
    }
    loadPlan();

    // Initialize Modals
    if (editMealModalRef.value) {
        mdbEditMealModal = new mdb.Modal(editMealModalRef.value);
    }
    if (editWorkoutModalRef.value) {
        mdbEditWorkoutModal = new mdb.Modal(editWorkoutModalRef.value);
    }
     if (workoutModalRef.value) {
        mdbWorkoutModal = new mdb.Modal(workoutModalRef.value);
    }
    if (mealsModalRef.value) mdbMealsModal = new mdb.Modal(mealsModalRef.value);
    if (profileModalRef.value) mdbProfileModal = new mdb.Modal(profileModalRef.value);

    const savedCompletion = localStorage.getItem('workoutCompletionStatus');
    if (savedCompletion) {
        Object.assign(workoutCompletionStatus, JSON.parse(savedCompletion));
    }

       // Inizializzazione dei nuovi modali per l'IA
    if (loadingModalRef.value) mdbLoadingModal = new mdb.Modal(loadingModalRef.value);
    if (aiSuggestionsModalRef.value) mdbAiSuggestionsModal = new mdb.Modal(aiSuggestionsModalRef.value);
    if (premiumModalRef.value) premiumModalInstance = new mdb.Modal(premiumModalRef.value);
});
</script>

