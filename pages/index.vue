<template>
  <section style="padding: 2rem;">
    <h1 style="font-size: 2rem; font-weight: bold; margin-bottom: 1rem;">
      VCE Median Study Scores (2022)
    </h1>

    <!-- Filters -->
    <div style="margin-bottom: 2rem; display: flex; flex-wrap: wrap; gap: 1rem;">
      <!-- School Name Filter -->
      <div style="flex: 1 1 250px;">
        <label style="font-weight: 500;">Search by School Name</label>
        <input
          v-model="filterName"
          type="text"
          placeholder="e.g. Melbourne High"
          style="padding: 0.5rem; width: 100%; border: 1px solid #ccc; border-radius: 4px;"
        />
      </div>

      <!-- Locality Filter -->
      <div style="flex: 1 1 250px;">
        <label style="font-weight: 500;">Filter by Locality</label>
        <select
          v-model="filterLocality"
          style="padding: 0.5rem; width: 100%; border: 1px solid #ccc; border-radius: 4px;"
        >
          <option value="">All</option>
          <option v-for="loc in uniqueLocalities" :key="loc" :value="loc">{{ loc }}</option>
        </select>
      </div>

      <!-- Minimum Score Filter -->
      <div style="flex: 1 1 200px;">
        <label style="font-weight: 500;">Minimum Median Score</label>
        <select
          v-model.number="filterScore"
          style="padding: 0.5rem; width: 100%; border: 1px solid #ccc; border-radius: 4px;"
        >
          <option :value="0">All</option>
          <option v-for="score in scoreOptions" :key="score" :value="score">
            ≥ {{ score }}
          </option>
        </select>
      </div>

      <!-- Sort Order -->
      <div style="flex: 1 1 200px;">
        <label style="font-weight: 500;">Sort by Score</label>
        <select
          v-model="sortOrder"
          style="padding: 0.5rem; width: 100%; border: 1px solid #ccc; border-radius: 4px;"
        >
          <option value="">None</option>
          <option value="desc">High → Low</option>
          <option value="asc">Low → High</option>
        </select>
      </div>
    </div>

    <!-- Table -->
    <table style="width: 100%; border-collapse: collapse;">
      <thead>
        <tr style="background-color: #f2f2f2;">
          <th style="text-align: left; padding: 0.5rem;">School</th>
          <th style="text-align: left; padding: 0.5rem;">Locality</th>
          <th style="text-align: center; padding: 0.5rem;">Median Score</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="school in sortedSchools"
          :key="school.School + school.Locality"
          style="border-bottom: 1px solid #e0e0e0;"
        >
          <td style="padding: 0.5rem;">{{ school.School }}</td>
          <td style="padding: 0.5rem;">{{ school.Locality }}</td>
          <td style="text-align: center; padding: 0.5rem;">
            {{ school['Median VCE study score'] }}
          </td>
        </tr>
      </tbody>
    </table>

    <p v-if="sortedSchools.length === 0" style="margin-top: 1rem; color: #a00;">
      No schools match your search.
    </p>
  </section>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const schools = ref([])
const filterName = ref('')
const filterLocality = ref('')
const filterScore = ref(0)
const sortOrder = ref('') // '', 'asc', or 'desc'

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
</script>
