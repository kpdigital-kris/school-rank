<template>
  <section class="section">
    <div class="container">
      <h1 class="title">VCE Median Study Scores (2022)</h1>

      <!-- Comparison Panel -->
      <div v-if="comparedSchools.length" class="box mb-5">
        <h2 class="subtitle">Comparison View</h2>
        <table class="table is-striped is-hoverable is-fullwidth">
          <thead>
            <tr>
              <th>School</th>
              <th>Locality</th>
              <th class="has-text-centered">Median Score</th>
              <th class="has-text-centered"></th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="school in comparedSchools"
              :key="school.School + school.Locality"
              :class="{ 'has-background-success-light': isTopScore(school) }"
            >
              <td>{{ school.School }}</td>
              <td>{{ school.Locality }}</td>
              <td class="has-text-centered">{{ school['Median VCE study score'] }}</td>
              <td class="has-text-centered">
                <button class="button is-small is-danger is-light" @click="removeCompare(school)">
                  Remove
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Filters -->
      <div class="columns is-multiline mb-5">
        <div class="column is-one-quarter">
          <div class="field">
            <label class="label">Search by School Name</label>
            <div class="control">
              <input
                class="input"
                type="text"
                v-model="filterName"
                placeholder="e.g. Melbourne High"
              />
            </div>
          </div>
        </div>

        <div class="column is-one-quarter">
          <div class="field">
            <label class="label">Filter by Locality</label>
            <div class="control">
              <div class="select is-fullwidth">
                <select v-model="filterLocality">
                  <option value="">All</option>
                  <option v-for="loc in uniqueLocalities" :key="loc" :value="loc">{{ loc }}</option>
                </select>
              </div>
            </div>
          </div>
        </div>

        <div class="column is-one-quarter">
          <div class="field">
            <label class="label">Minimum Median Score</label>
            <div class="control">
              <div class="select is-fullwidth">
                <select v-model.number="filterScore">
                  <option :value="0">All</option>
                  <option v-for="score in scoreOptions" :key="score" :value="score">≥ {{ score }}</option>
                </select>
              </div>
            </div>
          </div>
        </div>

        <div class="column is-one-quarter">
          <div class="field">
            <label class="label">Sort by Score</label>
            <div class="control">
              <div class="select is-fullwidth">
                <select v-model="sortOrder">
                  <option value="">None</option>
                  <option value="desc">High → Low</option>
                  <option value="asc">Low → High</option>
                </select>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- School Table -->
      <table class="table is-bordered is-striped is-hoverable is-fullwidth">
        <thead>
          <tr>
            <th></th>
            <th>School</th>
            <th>Locality</th>
            <th class="has-text-centered">Median Score</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="school in sortedSchools"
            :key="school.School + school.Locality"
          >
            <td class="has-text-centered">
              <input
                type="checkbox"
                :checked="isCompared(school)"
                @change="toggleCompare(school)"
              />
            </td>
            <td>{{ school.School }}</td>
            <td>{{ school.Locality }}</td>
            <td class="has-text-centered">{{ school['Median VCE study score'] }}</td>
          </tr>
        </tbody>
      </table>

      <div v-if="sortedSchools.length === 0" class="notification is-warning">
        No schools match your search.
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const schools = ref([])
const filterName = ref('')
const filterLocality = ref('')
const filterScore = ref(0)
const sortOrder = ref('')
const comparedSchools = ref([])

onMounted(async () => {
  try {
    const res = await fetch('/top_vce_schools_2022.json')
    schools.value = await res.json()
  } catch (err) {
    console.error('Failed to load school data:', err)
  }
})

const filteredSchools = computed(() =>
  schools.value.filter((s) => {
    const matchesName = s.School.toLowerCase().includes(filterName.value.toLowerCase())
    const matchesLocality = filterLocality.value ? s.Locality === filterLocality.value : true
    const matchesScore = s['Median VCE study score'] >= filterScore.value
    return matchesName && matchesLocality && matchesScore
  })
)

const sortedSchools = computed(() => {
  if (!sortOrder.value) return filteredSchools.value
  return [...filteredSchools.value].sort((a, b) => {
    const aScore = a['Median VCE study score']
    const bScore = b['Median VCE study score']
    return sortOrder.value === 'asc' ? aScore - bScore : bScore - aScore
  })
})

const uniqueLocalities = computed(() => {
  const set = new Set(schools.value.map((s) => s.Locality).sort())
  return Array.from(set)
})

const scoreOptions = computed(() => {
  const set = new Set(schools.value.map((s) => Math.floor(s['Median VCE study score'])))
  return Array.from(set).sort((a, b) => b - a)
})

function toggleCompare(school) {
  const exists = comparedSchools.value.find(
    (s) => s.School === school.School && s.Locality === school.Locality
  )
  if (exists) {
    comparedSchools.value = comparedSchools.value.filter(
      (s) => s.School !== school.School || s.Locality !== school.Locality
    )
  } else {
    comparedSchools.value.push(school)
  }
}

function removeCompare(school) {
  comparedSchools.value = comparedSchools.value.filter(
    (s) => s.School !== school.School || s.Locality !== school.Locality
  )
}

function isCompared(school) {
  return comparedSchools.value.some(
    (s) => s.School === school.School && s.Locality === school.Locality
  )
}

const topScore = computed(() => {
  if (!comparedSchools.value.length) return null
  return Math.max(...comparedSchools.value.map(s => s['Median VCE study score']))
})

function isTopScore(school) {
  return school['Median VCE study score'] === topScore.value
}
</script>
