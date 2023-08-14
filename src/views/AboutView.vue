<script setup lang="ts">
import DropdownSelect from '../components/DropdownSelect.vue'
import HeaderS from '../components/HeaderS.vue'
import SearchInput from '../components/SearchInput.vue'
import SideBar from '../components/SideBar.vue'
import EmployeeTable from '../components/EmployeeTable.vue'
import PaginationE from '../components/PaginationE.vue'
import { onMounted, ref } from 'vue'
import { useHttpApi } from '@/stores/useApiWeb';
import CommonConsts from '@/constants'
import { useAuthStore } from '../stores/authStore';

interface Employee {
  id: number;
  nombre: string;
  correo: string;
  cargo: string;
  departamento: string;
  oficina: string;
  estadoCuenta: string;
  active: boolean;
}

const store = useAuthStore();
const token = store.token;

let loading = ref(true);
let page = ref(1);
let pageSize = ref(10);
let employees: any;
let pagesNumber = ref(1);
let total = ref(1);

let dataLoaded = ref(false);

const api = useHttpApi();

const getUniqueCargos = (employees: Employee[]): string[] => {
  const cargos = employees.map(emp => emp.cargo);
  return [...new Set(cargos)];
}

const getEmployees = async () => {
  loading.value = true;

  const dataObject = JSON.stringify("");

  const link = `${CommonConsts.API_URL}/empleados?limit=${pageSize.value}&page=${page.value}`;

  const bearerToken = token as string;

  try {
    const response = await api.createGetQuery(link, bearerToken, dataObject);
    employees = response.data;
    employees.map((item: Employee) => {
      item.active = false;
    })
    const cargos = getUniqueCargos(employees);
    store.setCharges(cargos);
    total.value = response.total;
    pagesNumber.value = Math.ceil(response.total / pageSize.value);
    loading.value = false;
  } catch (error) {
    console.log(error);
  }
};

const handleActualpage = async (pageNew: number) => {
  dataLoaded.value = false;
  page.value = pageNew;
  await getEmployees();
  dataLoaded.value = true;
}

const handleActualLimit = async (newLimit: number) => {
  dataLoaded.value = false;
  pageSize.value = newLimit;
  page.value = 1;
  await getEmployees();
  dataLoaded.value = true;
}

const sortedEmployees = async (fieldName: keyof Employee) => {
  dataLoaded.value = false;
  loading.value = true;
  let employeeSorted = await employees.sort((a: Employee, b: Employee) => {
    if (a[fieldName] > b[fieldName]) {
      return 1;
    } else if (a[fieldName] < b[fieldName]) {
      return -1;
    } else {
      return 0;
    }
  });
  employees = employeeSorted;
  dataLoaded.value = true;
  loading.value = false;
}

const setActivationForCargo = async (cargoValue: string) => {
  dataLoaded.value = false;
  loading.value = true;
  await employees.forEach((employee: Employee) => {
    if (employee.cargo === cargoValue) {
      employee.active = true;
    } else {
      employee.active = false;
    }
  });
  dataLoaded.value = true;
  loading.value = false;
};

onMounted(async () => {
  await getEmployees()
  dataLoaded.value = true;
})
</script>

<template>
  <section class="flex bg-[#F8F8F8]">
    <side-bar />
    <div class="w-full md:h-screen">
      <header-s />
      <div class="pt-8 px-6 pb-8 md:pb-0 max-h-[calc(100vh-111px)] overflow-auto">
        <div v-if="!loading" class="bg-white p-6 rounded-2xl">
          <div class="flex flex-col md:flex-row gap-5 md:items-center justify-between mb-6">
            <div>
              <h3 class="text-[#111827] font-['Manrope'] text-2xl font-bold mb-2">{{ CommonConsts.home.employees }}</h3>
              <p class="text-[#687588] font-['Manrope'] text-sm font-medium">
                {{ CommonConsts.home.employeeText }}
              </p>
            </div>
            <div class="flex items-center gap-[20px]">
              <div class="flex items-center gap-[20px] w-full md:w-fit">
                <button
                  class="h-[48px] w-full md:w-fit px-6 border text-[#111827] border-[#111827] flex items-center justify-center gap-2 rounded-[10px] text-sm font-bold">
                  <span class="material-icons text-[20px]">{{ CommonConsts.home.downloadIcon }}</span>
                  <span class="font-['Manrope']">{{ CommonConsts.home.download }}</span>
                </button>
                <button
                  class="h-[48px] w-full md:w-fit text-white px-6 bg-[#111827] flex items-center justify-center gap-2 rounded-[10px] text-sm font-bold">
                  <span class="material-icons text-[20px]">{{ CommonConsts.home.newIcon }}</span>
                  <span class="font-['Manrope']">{{ CommonConsts.home.new }}</span>
                </button>
              </div>
            </div>
          </div>
          <div class="mb-6">
            <div class="flex md:items-center flex-col md:flex-row gap-5">
              <div class="w-full md:w-8/12">
                <search-input />
              </div>
              <div class="w-full md:w-4/12">
                <dropdown-select @activate-cargo="setActivationForCargo" />
              </div>
            </div>
          </div>
          <div>
            <div class="overflow-auto">
              <div class="min-w-[800px]">
                <EmployeeTable v-if="dataLoaded" :employee-list="employees" @sorting-list="sortedEmployees" />
              </div>
            </div>
            <div class="flex flex-col gap-5 md:flex-row items-center justify-between pt-6">
              <pagination-e :total="total" :page="page" :pages="pagesNumber" :per-page="pageSize"
                @custom-event="handleActualpage" @new-limit="handleActualLimit" />
            </div>
          </div>
        </div>
        <div v-else class="bg-white p-6 rounded-2xl">
          <div class="flex flex-col md:flex-row gap-5 md:items-center justify-between mb-6">
            <div>
              <h3 class="text-[#111827] font-['Manrope'] text-2xl font-bold mb-2">Empleados</h3>
            </div>
            <div class="flex items-center gap-[20px]">
              <div class="w-[100px] h-[34px] bg-[#F8F8F8] animate-pulse rounded-full loading"></div>
            </div>
          </div>
          <div class="mb-6">
            <div class="flex items-center gap-4 loading">
              <div class="w-full h-[34px] bg-[#F8F8F8] animate-pulse rounded-full"></div>
              <div class="w-full h-[34px] bg-[#F8F8F8] animate-pulse rounded-full"></div>
              <div class="w-full h-[34px] bg-[#F8F8F8] animate-pulse rounded-full"></div>
            </div>
          </div>
          <div>
            <div class="overflow-auto">
              <div class="min-w-[800px]">
                <div>
                  <div class="flex items-center h-[56px] bg-[#FAFAFA] rounded-[10px] loading">
                    <div class="w-2/12 h-full flex items-center px-4">
                      <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                    </div>
                    <div class="w-2/12 h-full flex items-center px-4">
                      <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                    </div>
                    <div class="w-2/12 h-full flex items-center px-4">
                      <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                    </div>
                    <div class="w-2/12 h-full flex items-center px-4">
                      <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                    </div>
                    <div class="w-2/12 h-full flex items-center px-4">
                      <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                    </div>
                    <div class="w-2/12 h-full flex items-center justify-end gap-[10px] px-4">
                      <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                    </div>
                  </div>
                </div>
                <div class="overflow-auto max-h-[calc(100vh-499px)]">
                  <div>
                    <div class="flex items-center h-[56px] border-b border-[#F1F2F4] loading">
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center justify-end gap-[10px] px-4">
                        <div class="w-[20px] h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                        <div class="w-[20px] h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                        <div class="w-[20px] h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                    </div>
                  </div>
                  <div>
                    <div class="flex items-center h-[56px] border-b border-[#F1F2F4] loading">
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center justify-end gap-[10px] px-4">
                        <div class="w-[20px] h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                        <div class="w-[20px] h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                        <div class="w-[20px] h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                    </div>
                  </div>
                  <div>
                    <div class="flex items-center h-[56px] border-b border-[#F1F2F4] loading">
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center px-4">
                        <div class="w-full h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                      <div class="w-2/12 h-full flex items-center justify-end gap-[10px] px-4">
                        <div class="w-[20px] h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                        <div class="w-[20px] h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                        <div class="w-[20px] h-[20px] bg-[#F1F2F4] animate-pulse rounded-full"></div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped></style>