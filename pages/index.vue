<template>
  <section style="padding: 2rem;">
    <h1 style="font-size: 2rem; font-weight: bold; margin-bottom: 1rem;">
      VCE Median Study Scores
    </h1>

    <div style="margin-bottom: 1rem;">
      <label style="font-weight: 500;">Filter by school name:</label><br />
      <input
        v-model="filter"
        type="text"
        placeholder="e.g. Melbourne High"
        style="padding: 0.5rem; width: 100%; max-width: 400px; border: 1px solid #ccc; border-radius: 4px;"
      />
    </div>

    <table style="width: 100%; border-collapse: collapse;">
      <thead>
        <tr style="background-color: #f2f2f2;">
          <th style="text-align: left; padding: 0.5rem;">School Name</th>
          <th style="text-align: center; padding: 0.5rem;">Median Score</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="school in filteredSchools"
          :key="school.name"
          style="border-bottom: 1px solid #e0e0e0;"
        >
          <td style="padding: 0.5rem;">{{ school.name }}</td>
          <td style="padding: 0.5rem; text-align: center;">
            {{ school.score }}
          </td>
        </tr>
      </tbody>
    </table>

    <p v-if="filteredSchools.length === 0" style="margin-top: 1rem; color: #a00;">
      No schools match your search.
    </p>
  </section>
</template>

<script setup>
import { ref, computed } from 'vue'

const filter = ref('')

const schools = [
  { name: 'Melbourne High School', score: 35 },
  { name: 'Mac.Robertson Girls’ High School', score: 36 },
  { name: 'Haileybury College', score: 34 },
  { name: 'Camberwell Grammar School', score: 33 },
  { name: 'Wesley College', score: 32 },
  { name: 'Scotch College', score: 34 }
]

const filteredSchools = computed(() =>
  schools.filter((s) =>
    s.name.toLowerCase().includes(filter.value.toLowerCase())
  )
)
</script>