<template>
  <div class="container mt-4">
    <h2 class="mb-3">รายการพนักงาน</h2>

    <div class="mb-3">
      <button class="btn btn-primary" @click="openAddModal">เพิ่มพนักงาน+</button>
    </div>

    <table class="table table-bordered table-striped">
      <thead class="table-primary">
        <tr>
          <th>รหัสพนักงาน</th>
          <th>ชื่อ</th>
          <th>นามสกุล</th>
          <th>ที่อยู่</th>
          <th>โทรศัพท์</th>
          <th>รูปภาพ</th>
          <th>การจัดการ</th>
        </tr>
      </thead>
      <tbody>
  <tr v-for="emp in employees" :key="emp.emp_id">
    <td>{{ emp.emp_id }}</td>
    <td>{{ emp.first_name }}</td>
    <td>{{ emp.last_name }}</td>
    <td>{{ emp.address }}</td>
    <td>{{ emp.phone }}</td>


    <td>
      <img
        v-if="emp.image"
        :src="'http://localhost/my-vue-app/php_api/uploads/' + emp.image"
        width="60"
        class="rounded shadow-sm"
      />
      <span v-else class="text-muted">ไม่มีรูป</span>
    </td>

    <td>
      <button class="btn btn-warning btn-sm me-2" @click="openEditModal(emp)">
        แก้ไข
      </button>
      <button class="btn btn-danger btn-sm" @click="deleteEmployee(emp.emp_id)">
        ลบ
      </button>
    </td>
  </tr>
</tbody>
    </table>

    <div v-if="loading" class="text-center"><p>กำลังโหลดข้อมูล...</p></div>
    <div v-if="error" class="alert alert-danger">{{ error }}</div>

    <div class="modal fade" id="employeeModal" tabindex="-1">
      <div class="modal-dialog modal-md">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">{{ isEditMode ? "แก้ไขข้อมูลพนักงาน" : "เพิ่มพนักงานใหม่" }}</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="saveEmployee">
              
              <div class="mb-3">
                <label class="form-label">ชื่อ</label>
                <input v-model="editForm.first_name" type="text" class="form-control" required />
              </div>
              <div class="mb-3">
                <label class="form-label">นามสกุล</label>
                <input v-model="editForm.last_name" type="text" class="form-control" required />
              </div>
              <div class="mb-3">
                <label class="form-label">ที่อยู่</label>
                <input v-model="editForm.address" type="text" class="form-control" required />
              </div>
              <div class="mb-3">
                <label class="form-label">โทรศัพท์</label>
                <input v-model="editForm.phone" type="text" class="form-control" required />
              </div>
              <div class="mb-3">
                <label class="form-label">รูปภาพ</label>
                <input type="file" @change="handleFileUpload" class="form-control" :required="!isEditMode" />
                <div v-if="isEditMode && editForm.image" class="mt-2">
                  <small class="text-muted d-block">รูปปัจจุบัน:</small>
                  <img :src="'http://localhost/my-vue-app/php_api/uploads/' + editForm.image" width="100" class="mt-1 rounded" />
                </div>
              </div>
              <div class="modal-footer px-0 pb-0">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">ยกเลิก</button>
                <button type="submit" class="btn btn-success">
                  {{ isEditMode ? "บันทึกการแก้ไข" : "บันทึกข้อมูล" }}
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";

export default {
  name: "EmployeeList",
  setup() {
    const employees = ref([]);
    const loading = ref(true);
    const error = ref(null);
    const isEditMode = ref(false);
    const editForm = ref({
      emp_id: null,
      first_name: "",
      last_name: "",
      address: "",
      phone: "",
      image: ""
    });
    const newImageFile = ref(null);
    let modalInstance = null;

    // โหลดข้อมูลพนักงาน
    const fetchEmployees = async () => {
      try {
        const res = await fetch("http://localhost/my-vue-app/php_api/employee_edit.php");
        const data = await res.json();
        employees.value = data.success ? data.data : [];
      } catch (err) {
        error.value = "ไม่สามารถโหลดข้อมูลได้: " + err.message;
      } finally {
        loading.value = false;
      }
    };

    const openAddModal = () => {
      isEditMode.value = false;
      editForm.value = { emp_id: "", first_name: "", last_name: "", address: "", phone: "", image: "" };
      newImageFile.value = null;
      showModal();
    };

    const openEditModal = (emp) => {
      isEditMode.value = true;
      editForm.value = { ...emp };
      newImageFile.value = null;
      showModal();
    };

    const showModal = () => {
      const modalEl = document.getElementById("employeeModal");
      modalInstance = new window.bootstrap.Modal(modalEl);
      modalInstance.show();
      // Reset input file
      const fileInput = modalEl.querySelector('input[type="file"]');
      if (fileInput) fileInput.value = "";
    };

    const handleFileUpload = (event) => {
      newImageFile.value = event.target.files[0];
    };

    const saveEmployee = async () => {
      const formData = new FormData();
      formData.append("action", isEditMode.value ? "update" : "add");
      formData.append("emp_id", editForm.value.emp_id);
      formData.append("first_name", editForm.value.first_name);
      formData.append("last_name", editForm.value.last_name);
      formData.append("address", editForm.value.address);
      formData.append("phone", editForm.value.phone);
      if (newImageFile.value) formData.append("image", newImageFile.value);

      try {
        const res = await fetch("http://localhost/my-vue-app/php_api/employee_edit.php", {
          method: "POST",
          body: formData
        });
        const result = await res.json();
        if (result.message) {
          alert(result.message);
          fetchEmployees();
          modalInstance.hide();
        } else {
          alert(result.error || "เกิดข้อผิดพลาด");
        }
      } catch (err) {
        alert("เชื่อมต่อ Server ล้มเหลว: " + err.message);
      }
    };

    const deleteEmployee = async (id) => {
      if (!confirm(`คุณแน่ใจหรือไม่ที่จะลบพนักงานรหัส ${id}?`)) return;

      const formData = new FormData();
      formData.append("action", "delete");
      formData.append("emp_id", id);

      try {
        const res = await fetch("http://localhost/my-vue-app/php_api/employee_edit.php", {
          method: "POST",
          body: formData
        });
        const result = await res.json();
        if (result.message) {
          alert(result.message);
          employees.value = employees.value.filter((e) => e.emp_id !== id);
        } else {
          alert(result.error);
        }
      } catch (err) {
        alert(err.message);
      }
    };

    onMounted(fetchEmployees);

    return {
      employees,
      loading,
      error,
      editForm,
      isEditMode,
      openAddModal,
      openEditModal,
      handleFileUpload,
      saveEmployee,
      deleteEmployee
    };
  }
};
</script>