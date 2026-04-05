<template>
  <div class="container py-5">
    <div class="row justify-content-center">
      <div class="col-12 col-md-8 col-lg-6">

        <div class="card shadow-lg border-0 rounded-4">

          <!-- Header -->
          <div class="card-header text-center text-white rounded-top-4"
            style="background: linear-gradient(135deg, #0d6efd, #0dcaf0);">
            <h4 class="mb-1 fw-bold">📦 ระบบเบิกอุปกรณ์</h4>
            <small>กรอกข้อมูลเพื่อส่งเข้า n8n Webhook</small>
          </div>

          <!-- Body -->
          <div class="card-body p-4">

            <!-- Alert -->
            <div v-if="status.message" :class="`alert alert-${status.type} alert-dismissible fade show`">
              {{ status.message }}
              <button type="button" class="btn-close" @click="status.message = ''"></button>
            </div>

            <!-- Form -->
            <form @submit.prevent="submitForm">

              <div class="mb-3">
                <label class="form-label">ชื่อ-นามสกุล *</label>
                <input type="text" class="form-control" v-model="form.fullname" required>
              </div>

              <div class="mb-3">
                <label class="form-label">แผนก *</label>
                <input type="text" class="form-control" v-model="form.department" required>
              </div>

              <div class="mb-3">
                <label class="form-label">รายการอุปกรณ์ *</label>
                <input type="text" class="form-control" v-model="form.product_name" required>
              </div>

              <div class="mb-3">
                <label class="form-label">จำนวน *</label>
                <input type="number" class="form-control" v-model="form.qty" required>
              </div>

              <button class="btn btn-primary w-100 fw-bold" :disabled="loading">
                <span v-if="loading" class="spinner-border spinner-border-sm me-2"></span>
                {{ loading ? 'กำลังส่งข้อมูล...' : '💾 บันทึกข้อมูล' }}
              </button>

            </form>

          </div>

        </div>

      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from "vue";

const form = reactive({
  fullname: "",
  department: "",
  product_name: "",
  qty: ""
});

const loading = ref(false);

const status = reactive({
  message: "",
  type: ""
});

// 🔥 ใช้ tdate ให้ตรง n8n
const getTimestamp = () => {
  return new Date().toLocaleString("sv-SE", {
    timeZone: "Asia/Bangkok"
  }).replace("T", " ");
};

const submitForm = async () => {
  loading.value = true;
  status.message = "";

  try {
    const response = await fetch("http://localhost:5678/webhook/add-product", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        tdate: getTimestamp(), // 🔥 ตรงกับ n8n
        "ชื่อ-นามสกุล": form.fullname,
        "แผนก": form.department,
        "รายชื่ออุปกรณ์": form.product_name,
        "จำนวนที่เบิก": Number(form.qty)
      })
    });

    if (!response.ok) {
      throw new Error("Error");
    }

    const result = await response.json();
    console.log("SEND:", result);

    status.message = "✅ บันทึกข้อมูลเรียบร้อย!";
    status.type = "success";

    // reset form
    form.fullname = "";
    form.department = "";
    form.product_name = "";
    form.qty = "";

  } catch (error) {
    console.error(error);
    status.message = "❌ เกิดข้อผิดพลาด กรุณาลองใหม่";
    status.type = "danger";
  } finally {
    loading.value = false;
  }
};
</script>

<style scoped>
.card {
  transition: 0.3s;
}

.card:hover {
  transform: translateY(-5px);
}

.form-control:focus {
  box-shadow: 0 0 0 0.2rem rgba(13, 110, 253, 0.2);
  border-color: #0d6efd;
}
</style>