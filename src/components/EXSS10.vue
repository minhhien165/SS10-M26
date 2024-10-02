<template>
  <div>
    <div class="w-[100%] m-auto mt-4 h-[100vh]">
      <main class="main">
        <header class="d-flex justify-content-between mb-3">
          <h3>Nhân viên</h3>
          <button @click="showForm = true" class="btn btn-primary">
            Thêm mới nhân viên
          </button>
        </header>
        <div class="d-flex align-items-center justify-content-end gap-2 mb-3">
          <input
            style="width: 350px"
            type="text"
            class="form-control"
            placeholder="Tìm kiếm theo email"
          />
          <i class="fa-solid fa-arrows-rotate" title="Refresh"></i>
        </div>

        <table class="table table-bordered table-hover table-striped">
          <thead>
            <tr>
              <th>STT</th>
              <th>Họ và tên</th>
              <th>Ngày sinh</th>
              <th>Email</th>
              <th>Địa chỉ</th>
              <th>Trạng thái</th>
              <th colspan="2">Chức năng</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(employee, index) in employees" :key="employee.id">
              <td>{{ index + 1 }}</td>
              <td>{{ employee.name }}</td>
              <td>{{ employee.birthDate }}</td>
              <td>{{ employee.email }}</td>
              <td>{{ employee.address }}</td>
              <td>
                <div class="d-flex align-items-center gap-2">
                  <div class="status status-active"></div>
                  <span> Đang hoạt động</span>
                </div>
              </td>
              <td>
                <span
                  class="button button-block"
                  @click="confirmBlock(employee)"
                >
                  {{ employee.status === "active" ? "Chặn" : "Bỏ chặn" }}
                </span>
              </td>
              <td>
                <span
                  class="button button-edit"
                  @click="editEmployee(employee)"
                >
                  Sửa
                </span>
              </td>
              <td>
                <span
                  class="button button-delete"
                  @click="confirmDelete(employee)"
                  >Xóa</span
                >
              </td>
            </tr>
          </tbody>
        </table>
        <footer class="d-flex justify-content-end align-items-center gap-3">
          <select class="form-select">
            <option selected>Hiển thị 10 bản ghi trên trang</option>
            <option>Hiển thị 20 bản ghi trên trang</option>
            <option>Hiển thị 50 bản ghi trên trang</option>
            <option>Hiển thị 100 bản ghi trên trang</option>
          </select>
          <ul class="pagination">
            <li class="page-item">
              <a class="page-link" href="#">Previous</a>
            </li>
            <li class="page-item"><a class="page-link" href="#">1</a></li>
            <li class="page-item"><a class="page-link" href="#">2</a></li>
            <li class="page-item"><a class="page-link" href="#">3</a></li>
            <li class="page-item"><a class="page-link" href="#">Next</a></li>
          </ul>
        </footer>
      </main>
    </div>

    <div class="overlay" v-if="showForm">
      <form class="form" @submit.prevent="saveEmployee">
        <div class="d-flex justify-content-between align-items-center">
          <h4>
            {{ isEditing ? "Sửa thông tin nhân viên" : "Thêm mới nhân viên" }}
          </h4>
          <i @click="closeForm" class="fa-solid fa-xmark"></i>
        </div>
        <div>
          <label class="form-label" for="userName">Họ và tên</label>
          <input
            id="userName"
            type="text"
            class="form-control"
            v-model="currentEmployee.name"
          />
          <div v-if="validationErrors.name" class="form-text error">
            {{ validationErrors.name }}
          </div>
        </div>
        <div>
          <label class="form-label" for="dateOfBirth">Ngày sinh</label>
          <input
            id="dateOfBirth"
            type="date"
            class="form-control"
            v-model="currentEmployee.birthDate"
          />
          <div v-if="validationErrors.birthDate" class="form-text error">
            {{ validationErrors.birthDate }}
          </div>
        </div>
        <div>
          <label class="form-label" for="email">Email</label>
          <input
            id="email"
            type="text"
            class="form-control"
            v-model="currentEmployee.email"
          />
          <div v-if="validationErrors.email" class="form-text error">
            {{ validationErrors.email }}
          </div>
        </div>
        <div>
          <label class="form-label" for="address">Địa chỉ</label>
          <textarea
            class="form-control"
            id="address"
            rows="3"
            v-model="currentEmployee.address"
          ></textarea>
        </div>
        <div>
          <button class="w-100 btn btn-primary" type="submit">
            {{ isEditing ? "Cập nhật" : "Thêm mới" }}
          </button>
        </div>
      </form>
    </div>

    <div class="overlay" v-if="showConfirmBlock">
      <div class="modal-custom">
        <div class="modal-title">
          <h4>Cảnh báo</h4>
          <i @click="cancelBlock" class="fa-solid fa-xmark"></i>
        </div>
        <div class="modal-body-custom">
          <span
            >Bạn có chắc chắn muốn
            {{ confirmAction === "block" ? "chặn" : "bỏ chặn" }} tài khoản
            này?</span
          >
        </div>
        <div class="modal-footer-custom">
          <button class="btn btn-light" @click="cancelBlock">Hủy</button>
          <button class="btn btn-danger" @click="executeBlockOrUnblock">
            Xác nhận
          </button>
        </div>
      </div>
    </div>

    <div class="overlay" v-if="showConfirmDelete">
      <div class="modal-custom">
        <div class="modal-title">
          <h4>Cảnh báo</h4>
          <i @click="showConfirmDelete = false" class="fa-solid fa-xmark"></i>
        </div>
        <div class="modal-body-custom">
          <span>Bạn có chắc chắn muốn xóa tài khoản này?</span>
        </div>
        <div class="modal-footer-custom">
          <button class="btn btn-light" @click="showConfirmDelete = false">
            Hủy
          </button>
          <button class="btn btn-danger" @click="executeDelete">
            Xác nhận
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const employees = ref(JSON.parse(localStorage.getItem("employees")) || []);
const showForm = ref(false);
const showConfirmBlock = ref(false);
const confirmAction = ref("");
const currentEmployee = ref(null);
const validationErrors = ref({});
const isEditing = ref(false);
const showConfirmDelete = ref(false);
const employeeToDelete = ref(null);

const newEmployee = ref({
  name: "",
  birthDate: "",
  email: "",
  address: "",
});

const addEmployee = () => {
  if (
    !newEmployee.value.name ||
    !newEmployee.value.birthDate ||
    !newEmployee.value.email
  ) {
    return;
  }
  const id = employees.value.length + 1;
  employees.value.push({
    id,
    ...newEmployee.value,
    status: "active",
  });
  localStorage.setItem("employees", JSON.stringify(employees.value));

  newEmployee.value = {
    name: "",
    birthDate: "",
    email: "",
    address: "",
  };
  showForm.value = false;
};

const editEmployee = (employee) => {
  currentEmployee.value = { ...employee };
  isEditing.value = true;
  showForm.value = true;
};

const saveEmployee = () => {
  validationErrors.value = {};

  if (!currentEmployee.value.name) {
    validationErrors.value.name = "Họ và tên không được để trống.";
  }
  if (!currentEmployee.value.email) {
    validationErrors.value.email = "Email không được để trống.";
  } else if (!/\S+@\S+\.\S+/.test(currentEmployee.value.email)) {
    validationErrors.value.email = "Email không hợp lệ.";
  }
  const today = new Date().toISOString().split("T")[0];
  if (currentEmployee.value.birthDate > today) {
    validationErrors.value.birthDate =
      "Ngày sinh không được lớn hơn ngày hiện tại.";
  }

  if (Object.keys(validationErrors.value).length > 0) {
    return;
  }

  if (isEditing.value) {
    const index = employees.value.findIndex(
      (emp) => emp.id === currentEmployee.value.id
    );
    employees.value[index] = { ...currentEmployee.value };
  } else {
    const id = employees.value.length + 1;
    employees.value.push({
      id,
      ...currentEmployee.value,
      status: "active",
    });
  }

  localStorage.setItem("employees", JSON.stringify(employees.value));

  closeForm();
};

const closeForm = () => {
  showForm.value = false;
  currentEmployee.value = {};
  isEditing.value = false;
};

const confirmBlock = (employee) => {
  currentEmployee.value = employee;
  confirmAction.value = employee.status === "active" ? "block" : "unblock";
  showConfirmBlock.value = true;
};
const executeBlockOrUnblock = () => {
  if (confirmAction.value === "block") {
    currentEmployee.value.status = "blocked";
  } else {
    currentEmployee.value.status = "active";
  }

  localStorage.setItem("employees", JSON.stringify(employees.value));

  showConfirmBlock.value = false;
  currentEmployee.value = null;
};

const cancelBlock = () => {
  showConfirmBlock.value = false;
};

const deleteEmployee = (employeeId) => {
  employees.value = employees.value.filter((emp) => emp.id !== employeeId);

  localStorage.setItem("employees", JSON.stringify(employees.value));
};

const confirmDelete = (employee) => {
  employeeToDelete.value = employee;
  showConfirmDelete.value = true;
};

const executeDelete = () => {
  deleteEmployee(employeeToDelete.value.id);
  showConfirmDelete.value = false;
};
</script>

<style scoped>
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
