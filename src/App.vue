<template>
  <div class="w-[80%] m-auto mt-4 h-[100vh]">
    <main class="main">
      <header class="d-flex justify-content-between mb-3">
        <h3>Nhân viên</h3>
        <button @click="openForm" class="btn btn-primary">Thêm mới nhân viên</button>
      </header>
      <div class="d-flex align-items-center justify-content-end gap-2 mb-3">
        <input v-model="searchTerm" @input="searchEmployee" style="width: 350px" type="text" class="form-control" placeholder="Tìm kiếm theo email" />
        <i class="fa-solid fa-arrows-rotate" @click="refreshPage" title="Refresh"></i>
      </div>

      <!-- Danh sách nhân viên -->
      <table class="table table-bordered table-hover table-striped" v-if="employees.length">
        <thead>
          <tr>
            <th>STT</th>
            <th>Họ và tên</th>
            <th>Ngày sinh</th>
            <th>Email</th>
            <th>Địa chỉ</th>
            <th>Trạng thái</th>
            <th colspan="3">Chức năng</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(employee, index) in filteredEmployees" :key="employee.id">
            <td>{{ index + 1 }}</td>
            <td>{{ employee.name }}</td>
            <td>{{ employee.dob }}</td>
            <td>{{ employee.email }}</td>
            <td>{{ employee.address }}</td>
            <td>
              <div style="display: flex; align-items: center; gap: 8px">
                <div :class="{'status': true, 'status-active': employee.isActive, 'status-stop': !employee.isActive}"></div>
                <span> {{ employee.isActive ? 'Đang hoạt động' : 'Đã chặn' }}</span>
              </div>
            </td>
            <td>
              <span @click="toggleBlock(employee)" class="button" :class="employee.isActive ? 'button-block' : 'button-edit'">{{ employee.isActive ? 'Chặn' : 'Bỏ chặn' }}</span>
            </td>
            <td>
              <span @click="editEmployee(employee)" class="button button-edit">Sửa</span>
            </td>
            <td>
              <span @click="confirmDelete(employee)" class="button button-delete">Xóa</span>
            </td>
          </tr>
        </tbody>
      </table>
      <p v-else>Không có kết quả tìm kiếm</p>
    </main>

    <!-- Form thêm mới nhân viên -->
    <div class="overlay" v-if="isFormOpen">
      <form @submit.prevent="submitForm" class="form">
        <div class="d-flex justify-content-between align-items-center">
          <h4>{{ isEditMode ? 'Chỉnh sửa nhân viên' : 'Thêm mới nhân viên' }}</h4>
          <i class="fa-solid fa-xmark" @click="closeForm"></i>
        </div>
        <div>
          <label class="form-label" for="userName">Họ và tên</label>
          <input v-model="form.name" id="userName" type="text" class="form-control" />
          <span v-if="errors.name" class="error">{{ errors.name }}</span>
        </div>
        <div>
          <label class="form-label" for="dateOfBirth">Ngày sinh</label>
          <input v-model="form.dob" id="dateOfBirth" type="date" class="form-control" />
          <span v-if="errors.dob" class="error">{{ errors.dob }}</span>
        </div>
        <div>
          <label class="form-label" for="email">Email</label>
          <input v-model="form.email" id="email" type="text" class="form-control" />
          <span v-if="errors.email" class="error">{{ errors.email }}</span>
        </div>
        <div>
          <label class="form-label" for="address">Địa chỉ</label>
          <textarea v-model="form.address" class="form-control" id="address" rows="3"></textarea>
        </div>
        <div>
          <button class="w-100 btn btn-primary">{{ isEditMode ? 'Cập nhật' : 'Thêm mới' }}</button>
        </div>
      </form>
    </div>

    <!-- Modal xác nhận chặn tài khoản -->
    <div class="overlay" v-if="isModalOpen">
      <div class="modal-custom">
        <div class="modal-title">
          <h4>Cảnh báo</h4>
          <i class="fa-solid fa-xmark" @click="closeModal"></i>
        </div>
        <div class="modal-body-custom">
          <span>Bạn có chắc chắn muốn {{ isBlocking ? 'chặn' : 'bỏ chặn' }} tài khoản này?</span>
        </div>
        <div class="modal-footer-custom">
          <button class="btn btn-light" @click="closeModal">Hủy</button>
          <button class="btn btn-danger" @click="confirmBlockToggle">{{ isBlocking ? 'Xác nhận' : 'Bỏ chặn' }}</button>
        </div>
      </div>
    </div>

    <!-- Modal xác nhận xóa tài khoản -->
    <div class="overlay" v-if="isDeleteModalOpen">
      <div class="modal-custom">
        <div class="modal-title">
          <h4>Cảnh báo</h4>
          <i class="fa-solid fa-xmark" @click="closeDeleteModal"></i>
        </div>
        <div class="modal-body-custom">
          <span>Bạn có chắc chắn muốn xóa tài khoản này?</span>
        </div>
        <div class="modal-footer-custom">
          <button class="btn btn-light" @click="closeDeleteModal">Hủy</button>
          <button class="btn btn-danger" @click="deleteEmployee">Xác nhận</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';

// State and variables
const employees = ref([]);
const isFormOpen = ref(false);
const isEditMode = ref(false);
const form = ref({ id: null, name: '', dob: '', email: '', address: '', isActive: true });
const errors = ref({ name: '', dob: '', email: '' });
const searchTerm = ref('');
const filteredEmployees = computed(() => {
  if (!searchTerm.value) return employees.value;
  return employees.value.filter(e => e.email.toLowerCase().includes(searchTerm.value.toLowerCase()));
});
const isModalOpen = ref(false);
const isBlocking = ref(false);
const employeeToToggle = ref(null);
const isDeleteModalOpen = ref(false);
const employeeToDelete = ref(null);

// Function to open form
const openForm = () => {
  isFormOpen.value = true;
  resetForm();
};

// Function to close form
const closeForm = () => {
  isFormOpen.value = false;
  resetForm();
};

// Function to reset form data
const resetForm = () => {
  form.value = { id: null, name: '', dob: '', email: '', address: '', isActive: true };
  errors.value = { name: '', dob: '', email: '' };
  isEditMode.value = false;
};

// Function to submit form
const submitForm = () => {
  if (!validateForm()) return;
  
  if (isEditMode.value) {
    // Update existing employee
    const index = employees.value.findIndex(e => e.id === form.value.id);
    if (index !== -1) {
      employees.value[index] = { ...form.value };
      saveToLocalStorage();
    }
  } else {
    // Add new employee
    form.value.id = Date.now();
    employees.value.push({ ...form.value });
    saveToLocalStorage();
  }
  
  closeForm();
};

// Validation function
const validateForm = () => {
  errors.value = { name: '', dob: '', email: '' };
  let isValid = true;
  
  if (!form.value.name) {
    errors.value.name = 'Họ và tên không được để trống';
    isValid = false;
  }
  if (!form.value.email || !/\S+@\S+\.\S+/.test(form.value.email)) {
    errors.value.email = 'Email không đúng định dạng';
    isValid = false;
  }
  if (new Date(form.value.dob) > new Date()) {
    errors.value.dob = 'Ngày sinh không được lớn hơn ngày hiện tại';
    isValid = false;
  }
  
  return isValid;
};

// Function to search employees by email
const searchEmployee = () => {
  filteredEmployees.value = employees.value.filter(e => e.email.includes(searchTerm.value));
};

// Function to refresh page
const refreshPage = () => {
  searchTerm.value = '';
  loadFromLocalStorage();
};

// Function to toggle block/unblock
const toggleBlock = (employee) => {
  isBlocking.value = employee.isActive;
  employeeToToggle.value = employee;
  isModalOpen.value = true;
};

// Function to confirm block/unblock
const confirmBlockToggle = () => {
  employeeToToggle.value.isActive = !isBlocking.value;
  saveToLocalStorage();
  closeModal();
};

// Function to close modal
const closeModal = () => {
  isModalOpen.value = false;
  employeeToToggle.value = null;
};

// Function to confirm delete
const confirmDelete = (employee) => {
  isDeleteModalOpen.value = true;
  employeeToDelete.value = employee;
};

// Function to delete employee
const deleteEmployee = () => {
  employees.value = employees.value.filter(e => e.id !== employeeToDelete.value.id);
  saveToLocalStorage();
  closeDeleteModal();
};

// Function to close delete modal
const closeDeleteModal = () => {
  isDeleteModalOpen.value = false;
  employeeToDelete.value = null;
};

// Function to load data from localStorage
const loadFromLocalStorage = () => {
  const data = JSON.parse(localStorage.getItem('employees')) || [];
  employees.value = data;
};

// Function to save data to localStorage
const saveToLocalStorage = () => {
  localStorage.setItem('employees', JSON.stringify(employees.value));
};

// On mounted, load data
onMounted(() => {
  loadFromLocalStorage();
});
</script>

<style>
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}

body {
  font-size: 14px;
  font-family: sans-serif;
}

.main {
  margin: 50px 100px;
}

.fa-arrows-rotate {
  font-size: 20px;
  cursor: pointer;
  color: gray;
}

.fa-arrows-rotate:hover {
  color: rgb(184, 180, 180);
  transform: rotate(20deg);
  transition: all 0.5s linear;
}

.button {
  padding: 4px 12px;
  line-height: 30px;
  border-radius: 4px;
  color: #fff;
  cursor: pointer;
}

.button-edit {
  background-color: orange;
}

.button-delete {
  background-color: red;
}

.button-block {
  background-color: green;
}

td:first-child,
td:nth-child(6),
td:nth-child(7) {
  text-align: center;
}

.form-select {
  width: 270px !important;
}

.overlay {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.form {
  background-color: #fff;
  width: 500px;
  padding: 20px 24px;
  border-radius: 4px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.form-label {
  font-weight: 600;
  margin-bottom: 8px;
}

.fa-xmark {
  font-size: 20px;
  cursor: pointer;
}

.error {
  color: red !important;
}

.status-container {
  border: 1px solid #dadada;
  padding: 4px 8px;
  border-radius: 4px;
}

.status {
  height: 10px;
  width: 10px;
  border-radius: 50%;
  border: 1px solid transparent;
}

.status-active {
  background-color: green;
}

.status-stop {
  background-color: red;
}

.modal-custom {
  background-color: #fff;
  padding: 20px 24px;
  border-radius: 4px;
  width: 400px;
}

.modal-title {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.modal-body-custom {
  padding: 20px 0;
}

.modal-footer-custom {
  display: flex;
  justify-content: end;
  gap: 8px;
}

.pagination {
  margin-bottom: 0;
}
</style>
