<script setup>
import { ref, onMounted, computed } from 'vue';  // Menambahkan computed dari Vue
import axios from 'axios';

// Ref untuk menyimpan data employees dan kontrak yang dipilih untuk modal
const employees = ref([]);
const selectedEmployee = ref(null);
const showModal = ref(false);
const showEditModal = ref(false);
const showDeleteConfirm = ref(false);
const employeeToDelete = ref(null);

// Ref untuk data employee yang sedang diedit
const editedEmployee = ref({
    id: null,
    name: '',
    nik: '',
    no_hp: '',
    alamat: ''
});

// Ref untuk pencarian
const searchQuery = ref('');

// Fungsi untuk mengambil data employee beserta kontraknya
const fetchEmployees = async () => {
    try {
        const response = await axios.get('http://localhost:80/api/employees');
        employees.value = response.data;
    } catch (error) {
        console.error('Error fetching employees:', error);
    }
};

// Panggil fetchEmployees saat komponen pertama kali dimuat
onMounted(fetchEmployees);

// Fungsi untuk membuka modal dengan detail kontrak
const openContractModal = (employee) => {
    selectedEmployee.value = employee;
    showModal.value = true;
};

// Fungsi untuk menutup modal
const closeModal = () => {
    showModal.value = false;
};

// Fungsi untuk membuka modal edit employee
const openEditModal = (employee) => {
    editedEmployee.value = { ...employee };  // Menyalin data employee yang dipilih
    showEditModal.value = true;
};

// Fungsi untuk menutup modal edit
const closeEditModal = () => {
    showEditModal.value = false;
};

// Fungsi untuk mengedit data employee
const editEmployee = async () => {
    try {
        // Mengirim request PUT untuk mengupdate data employee
        const response = await axios.put(`http://localhost:80/api/employees/${editedEmployee.value.id}`, editedEmployee.value);

        // Update data lokal setelah berhasil mengedit
        const index = employees.value.findIndex(emp => emp.id === editedEmployee.value.id);
        if (index !== -1) {
            employees.value[index] = response.data;
        }

        alert('Employee updated successfully!');
        closeEditModal();  // Tutup modal edit
    } catch (error) {
        console.error('Error editing employee:', error);
    }
};

// Fungsi untuk membuka konfirmasi sebelum menghapus employee
const confirmDelete = (employeeId) => {
    employeeToDelete.value = employeeId;
    showDeleteConfirm.value = true;
};

// Fungsi untuk menghapus data employee setelah konfirmasi
const deleteEmployee = async () => {
    try {
        await axios.delete(`http://localhost:80/api/employees/${employeeToDelete.value}`);
        employees.value = employees.value.filter(emp => emp.id !== employeeToDelete.value);
        alert('Employee deleted successfully!');
        showDeleteConfirm.value = false;  // Tutup konfirmasi hapus
    } catch (error) {
        console.error('Error deleting employee:', error);
    }
};

// Fungsi untuk membatalkan penghapusan
const cancelDelete = () => {
    showDeleteConfirm.value = false;
};

// Fungsi untuk mencari employee berdasarkan nama atau NIK
const filteredEmployees = computed(() => {
    const query = searchQuery.value.toLowerCase();
    return employees.value.filter(employee => 
        employee.name.toLowerCase().includes(query) || 
        employee.nik.toLowerCase().includes(query)
    );
});
</script>

<template>
    <div class="container">
        <h1>List of Employees</h1>

        <!-- Kolom pencarian -->
        <div class="search-bar">
            <input 
                type="text" 
                v-model="searchQuery" 
                placeholder="Search by name or NIK" 
                class="search-input" />
        </div>

        <!-- Tabel untuk menampilkan daftar employee beserta kontraknya -->
        <table class="employee-table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>NIK</th>
                    <th>No HP</th>
                    <th>Alamat</th>
                    <th>Contracts</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="employee in filteredEmployees" :key="employee.id">
                    <td>{{ employee.id }}</td>
                    <td>{{ employee.name }}</td>
                    <td>{{ employee.nik }}</td>
                    <td>{{ employee.no_hp }}</td>
                    <td>{{ employee.alamat }}</td>
                    <td>
                        <button @click="openContractModal(employee)" class="view-btn">View Contracts</button>
                    </td>
                    <td>
                        <!-- Tombol untuk CRUD action -->
                        <button class="action-btn" @click="openEditModal(employee)">Edit</button>
                        <button class="action-btn" @click="confirmDelete(employee.id)">Delete</button>
                    </td>
                </tr>
            </tbody>
        </table>

        <!-- Modal untuk menampilkan kontrak -->
        <div v-if="showModal" class="modal">
            <div class="modal-content">
                <span @click="closeModal" class="close-btn">&times;</span>
                <h2>Contracts for <span class="contract-employee-name">{{ selectedEmployee?.name }}</span></h2>
                <ul>
                    <li v-for="contract in selectedEmployee?.contracts" :key="contract.id">
                        <strong>{{ contract.type_kontrak }}</strong> |
                        {{ contract.start_date }} - {{ contract.end_date }}
                    </li>
                </ul>
            </div>
        </div>

        <!-- Modal Edit Employee -->
        <div v-if="showEditModal" class="modal">
            <div class="modal-content">
                <span @click="closeEditModal" class="close-btn">&times;</span>
                <h2>Edit Employee</h2>
                <form @submit.prevent="editEmployee">
                    <label for="name">Name</label>
                    <input type="text" id="name" v-model="editedEmployee.name" required />

                    <label for="nik">NIK</label>
                    <input type="text" id="nik" v-model="editedEmployee.nik" required />

                    <label for="no_hp">No HP</label>
                    <input type="text" id="no_hp" v-model="editedEmployee.no_hp" required />

                    <label for="alamat">Alamat</label>
                    <input type="text" id="alamat" v-model="editedEmployee.alamat" required />

                    <button type="submit">Save</button>
                </form>
            </div>
        </div>

        <!-- Modal Konfirmasi Hapus Employee -->
        <div v-if="showDeleteConfirm" class="modal">
            <div class="modal-delete-content">
                <span @click="cancelDelete" class="close-btn">&times;</span>
                <p>Are you sure you want to delete this employee <span class="delete-employee-name">{{ employees.find(emp => emp.id === employeeToDelete)?.name }}</span>?</p>
                <div class="delete-buttons-container">
                    <button @click="cancelDelete" class="cancel-delete-btn">Cancel</button>
                    <button @click="deleteEmployee" class="delete-btn">Yes, Delete</button>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.container {
    max-width: 1000px;
    margin: 0 auto;
}

.search-bar {
    margin-bottom: 20px;
    display: flex;
    justify-content: flex-end;
}

.search-input {
    padding: 8px;
    width: 100%;
    max-width: 300px;
    border-radius: 4px;
    border: 1px solid #ccc;
}

.employee-table {
    width: 100%;
    border-collapse: collapse;
    margin-bottom: 20px;
}

.contract-employee-name {
    color: darkgreen
}

th,
td {
    text-align: left;
    padding: 10px;
    border: 1px solid #ccc;
}

th {
    background-color: #f2f2f2;
}

button {
    padding: 8px 15px;
    margin: 5px;
    cursor: pointer;
}

.delete-buttons-container {
    display: flex;
    justify-content: space-between;
}

.delete-employee-name {
    font-weight: bold;
    color: #e74c3c;
}

.delete-btn {
    background-color: #e74c3c;
    color: white;
    border: none;
    border-radius: 4px;
}

.cancel-delete-btn {
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 4px;
}

.view-btn {
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 4px;
}

.action-btn {
    background-color: #f39c12;
    color: white;
    border: none;
    border-radius: 4px;
}

.action-btn:first-of-type {
    background-color: #2980b9;
}

.action-btn:last-of-type {
    background-color: #e74c3c;
}

.modal {
    display: block;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    z-index: 1000;
}

.modal-content {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    width: 100%;
    max-width: 500px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
}

.modal-delete-content {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    width: 100%;
    max-width: 400px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
}

.close-btn {
    position: absolute;
    top: 10px;
    right: 10px;
    font-size: 24px;
    color: #333;
    cursor: pointer;
}

h2 {
    text-align: center;
    margin-bottom: 30px;
    margin-top: -5px;
}

form {
    display: flex;
    flex-direction: column;
}

form label {
    margin-top: 10px;
}

form input {
    padding: 8px;
    margin-bottom: 10px;
    border-radius: 4px;
    border: 1px solid #ccc;
}

button[type="submit"] {
    background-color: #2980b9;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 10px;
}
</style>
