<template>
    <div class="container mt-5">
        <div class="row">
            <div class="col-md-8 offset-md-2">
                <h1 class="text-center">User Information Form</h1>
                <form @submit.prevent="submitForm">
                    <div class="row mb-3">
                        <div class="col-md-6 col-sm-6">
                            <label for="username" class="form-label">Username</label>
                            <input type="text" class="form-control" id="username"
                                @blur = "() => validateName(true)"
                                @input = "() => validateName(false)"
                                v-model="formData.username">
                            <div v-if="errors.username" class="text-danger">{{ errors.username }}</div>
                        </div>

                        <div class="col-md-6 col-sm-6">
                            <label for="password" class="form-label">Password</label>
                            <input type="password" class="form-control" id="password" 
                                @blur="() => validatePassword(true)"
                                @input="() => validatePassword(false)"
                                v-model="formData.password">
                            <div v-if="errors.password" class="text-danger">{{ errors.password }}</div>
                        </div>
                    </div>
                    <div class="row mb-3">
                        <div class="col-md-6 col-sm-6">
                            <div class="form-check">
                                <input type="checkbox" class="form-check-input" id="isAustralian"
                                    @change="validateResident(true)"
                                    v-model="formData.isAustralian">
                                <label class="form-check-label" for="isAustralian">Australian Resident?</label>
                            </div>
                            <div v-if="errors.resident" class="text-danger mt-1">{{ errors.resident }}</div>
                        </div>

                        <div class="col-md-6 col-sm-6">
                            <label for="gender" class="form-label">Gender</label>
                            <select class="form-select" id="gender" 
                                @blur="() => validateGender(true)"
                                @change="() => validateGender(false)"
                                v-model="formData.gender">
                                <option value="">Please select...</option>
                                <option value="male">Male</option>
                                <option value="female">Female</option>
                                <option value="other">Other</option>
                            </select>
                            <div v-if="errors.gender" class="text-danger">{{ errors.gender }}</div>
                        </div>
                    </div>
                    <div class="mb-3">
                        <label for="reason" class="form-label">Reason for joining</label>
                        <textarea class="form-control" id="reason" rows="3" 
                            @blur="() => validateReason(true)"
                            @input="() => validateReason(false)"
                            v-model="formData.reason"
                            placeholder="Please tell us why you want to join..."></textarea>
                        <div v-if="errors.reason" class="text-danger">{{ errors.reason }}</div>
                    </div>
                    <div class="text-center">
                        <button type="submit" class="btn btn-primary me-2">Submit</button>
                        <button type="button" class="btn btn-secondary" @click="clearForm">Clear</button>
                    </div>
                </form>

                <!-- PrimeVue DataTable with forced white background -->
                <div class="mt-5" v-if="submittedCards.length">
                    <DataTable 
                        :value="submittedCards" 
                        tableStyle="min-width: 50rem; background-color: white;"
                        class="white-table">
                        <Column field="username" header="Username"></Column>
                        <Column field="password" header="Password"></Column>
                        <Column field="isAustralian" header="Australian Resident">
                            <template #body="slotProps">
                                <span :class="slotProps.data.isAustralian ? 'text-success' : 'text-danger'">
                                    {{ slotProps.data.isAustralian ? 'true' : 'false' }}
                                </span>
                            </template>
                        </Column>
                        <Column field="gender" header="Gender">
                            <template #body="slotProps">
                                <span class="text-capitalize">{{ slotProps.data.gender }}</span>
                            </template>
                        </Column>
                        <Column field="reason" header="Reason">
                            <template #body="slotProps">
                                <span>{{ slotProps.data.reason }}</span>
                            </template>
                        </Column>
                    </DataTable>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
    import { ref } from 'vue';
    import DataTable from 'primevue/datatable';
    import Column from 'primevue/column';
    
    const formData = ref({
        username: '',
        password: '',
        isAustralian: false,
        reason: '',
        gender: ''
    });

    const submittedCards = ref([]);

    const submitForm = () => {
        validateName(true);
        validatePassword(true);
        validateResident(true);
        validateGender(true);
        validateReason(true);

        if(!errors.value.username && 
            !errors.value.password && 
            !errors.value.resident && 
            !errors.value.gender && 
            !errors.value.reason) {
            submittedCards.value.push({ ...formData.value });
            clearForm();
        }
    };

    const clearForm = () => {
        formData.value = {
            username: '',
            password: '',
            isAustralian: false,
            reason: '',
            gender: ''
        };
        // エラーもクリア
        errors.value = {
            username: null,
            password: null,
            resident: null,
            reason: null,
            gender: null,
        };
    };

    const errors = ref({
        username: null,
        password: null,
        resident: null,
        reason: null,
        gender: null,
    });

    const validateName = (blur) => {
        if (formData.value.username.length < 3) {
            if (blur) errors.value.username = "Name must be at least 3 characters";
        } else {
            errors.value.username = null;
        }
    };

    const validatePassword = (blur) => {
        const password = formData.value.password;
        const minLength = 8;
        const hasUppercase = /[A-Z]/.test(password);
        const hasLowercase = /[a-z]/.test(password);
        const hasNumber = /\d/.test(password);
        const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);

        if(password.length < minLength){
            if (blur) errors.value.password = `Password must be at least ${minLength} characters long.`
        } else if (!hasUppercase) {
            if (blur) errors.value.password = 'Password must contain at least one uppercase letter.'
        } else if (!hasLowercase) {
            if (blur) errors.value.password = 'Password must contain at least one lowercase letter.'
        } else if (!hasNumber) {
            if (blur) errors.value.password = 'Password must contain at least one number.'
        } else if (!hasSpecialChar) {
            if (blur) errors.value.password = 'Password must contain at least one special character.'
        } else {
            errors.value.password = null;
        }
    };

    const validateResident = (blur) => {
        errors.value.resident = null;
    };

    const validateGender = (blur) => {
        if (!formData.value.gender || formData.value.gender === '') {
            if (blur) errors.value.gender = "Please select your gender.";
        } else {
            errors.value.gender = null;
        }
    };

    const validateReason = (blur) => {
        const reason = formData.value.reason.trim();
        if (reason.length < 10) {
            if (blur) errors.value.reason = "Please provide at least 10 characters for your reason.";
        } else if (reason.length > 500) {
            if (blur) errors.value.reason = "Reason must be less than 500 characters.";
        } else {
            errors.value.reason = null;
        }
    };
</script>

<style scoped>
/* フォームのスタイリング */
.form-control, .form-select {
    background-color: white !important;
}

/* DataTableの強制的な白い背景設定 */
.white-table :deep(.p-datatable) {
    background-color: white !important;
    border: 1px solid #dee2e6;
    border-radius: 0.375rem;
}

.white-table :deep(.p-datatable-wrapper) {
    background-color: white !important;
}

.white-table :deep(.p-datatable-header-cell) {
    background-color: white !important;
    color: #495057 !important;
    font-weight: 600;
    border: 1px solid #dee2e6 !important;
    padding: 0.75rem;
}

.white-table :deep(.p-datatable-tbody > tr > td) {
    background-color: white !important;
    color: #212529 !important;
    border: 1px solid #dee2e6 !important;
    padding: 0.75rem;
}

.white-table :deep(.p-datatable-tbody > tr:nth-child(even) > td) {
    background-color: #f8f9fa !important;
}

.white-table :deep(.p-datatable-tbody > tr:hover > td) {
    background-color: #e9ecef !important;
}

/* Bootstrapカラークラス */
.text-success {
    color: #198754 !important;
}

.text-danger {
    color: #dc3545 !important;
}

.text-capitalize {
    text-transform: capitalize;
}
</style>