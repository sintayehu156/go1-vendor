
     <template>

      <div>
    <!-- {{salesdata.originalData}} -->
          <div :class="['head-layout', { collapsed: isSidebarCollapsed }]">
            <div class="head-content">
              <header class="border-b bg-white px-5 py-3.5 font-medium sm:px-5">
                Purchase Invoice
              </header>
              <!-- {{ field_filters  }} -->
              <div class="flex justify-between w-1/2 mt-4 ml-4">
                <div class="p-1 w-36 " v-for="fieldData in filter_data" :key="fieldData.fieldname">
              
                  <component
                    :is="getComponentType(fieldData)"
                    v-bind="getComponentProps(fieldData)"
                      v-model="field_filters[fieldData.fieldname]"
                  />
                </div>
                <div class="mt-1 ml-6">
              <Button :variant="'subtle'" theme="gray" size="sm" @click="restsetFunction"> Reset</Button>
             </div>
              </div>
            </div>
            
          </div>
          <div :class="['layout', { collapsed: isSidebarCollapsed }]">
            <LeftSidebar :isCollapsed="isSidebarCollapsed" @toggle="toggleSidebar" />
            <div class="main-content" v-if="columns_data && supplier_detail.data" >
              <ListView
                class="h-[485px]"
                :columns="columns_data"
                 :rows="supplier_detail.data"
                :options="{
                  getRowRoute: (row) => ({ name: 'Purchase Detail', params: { id: row.name } }),
                  selectable: true,
                  showTooltip: true, 
                  resizeColumn: true,
                  emptyState: {
                    title: 'No records found',
                  },
                }"
                row-key="name"
                @row-click="OpenClick"
              >
              <template #cell="{ item,row, column }">
                <div v-if="column.key === 'status'">
                  <Badge
                    v-bind="getStatusTheme(item)"
                    size="sm"
                    :label="item"
                  />
                </div>
                <div v-else-if="column.key === 'naming_series'">
                  <span class="text-black text-base" style="max-width: 170px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; display: block;">
                    {{ item }}
                  </span>
                </div>
                <div v-else>
                  <span class="font-small text-gray-700 text-base" style="max-width: 170px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; display: block;">{{ item }}</span>
                </div>
              </template>
     
            </ListView>
            <ListFooter
            :modelValue="pageLengthCount"
            :options="{ rowCount: supplier_detail.data.length, totalCount: totalRows }"
            @update:modelValue="pageLengthCount = $event"
            @loadMore="supplier_detail.fetch()"
          />
              <!-- <pagination :rows="rows" @update:paginatedRows="updatePaginatedRows" />  -->
            </div>
          </div>
        </div>
      
      </template>
      
      <script>
    import LeftSidebar from '@/components/Custom Layout/LeftSidebar.vue'
    import ListView from '@/components/ListView/ListView.vue'
    import ListFooter from 'frappe-ui/src/components/ListView/ListFooter.vue'
    import RefreshButton from '@/components/RefreshButton.vue'
    import { ref,watch,reactive } from 'vue'
    import { createResource, createListResource,FormControl,Select,DatePicker,Badge } from 'frappe-ui'
    import { useRouter } from 'vue-router';
    
    export default {
      components: {
        LeftSidebar,
        ListView,
        Badge,
        ListFooter,
        RefreshButton
        // Pagination
      },
      setup() {
        let order = ref("")
        let salesdata = ref("")
        const isSidebarCollapsed = ref(false)
        
        let supplier_detail = ref([])
        // const rows = ref([])
        // const paginatedRows = ref([]) 
        
        let columns_data = ref([])
        let filter_data = ref([])
        let field_filters = reactive({});
        const pageLengthCount = ref(20); // Default to 20 rows
        const totalRows = ref(0); 
        
        
        supplier_detail = createResource({
          url:'go1_vendor.api.get_purchaseorder',
          method:'Get',
          auto:true
        })
        supplier_detail.fetch()
        console.log('supplier',supplier_detail)

      // supplier_detail = createListResource({
      // doctype: 'Purchase Order',
      // fields:["*"],
      // filters:field_filters,
      // limit: pageLengthCount.value,
      // orderBy:'name',
      // auto:true
      // })
        order = createResource({
          url: 'go1_vendor.sales.get_purchase',
          method: 'get',
        })
  
        order.fetch().then((res) =>{
          const field = res.fields
    
          columns_data.value = []
          filter_data.value = []

          filter_data.value.push({fieldname:'name',fieldtype:'Data',options:null,label:'ID'})

          
          for(let fielddata of field){
            if(fielddata.in_list_view==1){
              columns_data.value.push({
                    label:fielddata.label,
                    key:fielddata.fieldname,
                    width:fielddata.width
              })

              console.log('coldata',columns_data)
            }
            if(fielddata.in_standard_filter == 1){
              // console.log("PPPPPPPPPPPP+++++++++++++++_______________",fielddata)
              filter_data.value.push(fielddata)

            }
           
          }
        } )
        //  console.log('columns',columns_data)
        
        // console.log('data',order)
        

    
        // const fetchorder = async () => {
        //   try {
        //     const data = await order.fetch()
        //     rows.value = data.map(row => ({
        //       ...row,
        //       total: String(row.total),
        //       item_name: row.items.length > 0 ? row.items[0].item_name : 'No items',
        //     }))
        //     console.log('Fetched data:', rows.value)
        //   } catch (error) {
        //     console.error('Error fetching data:', error)
        //   }
        // }
    
        const toggleSidebar = () => {
          isSidebarCollapsed.value = !isSidebarCollapsed.value
        }
    
        const router = useRouter()
    
        const OpenClick = (row) => {
          console.log('Row clicked:', row)
          if (row && row.name) {
            router.push({ name: 'Purchase Detail', params: { id: row.name } })
          } else {
            console.error('Row data is invalid:', row)
          }
        }
        watch(pageLengthCount, (newPageLength) => {
          console.log("aaaa",newPageLength)
          supplier_detail.limit = newPageLength;
          supplier_detail.fetch(); // Re-fetch the data with the updated page length
        });
        
    
        // const updatePaginatedRows = (newPaginatedRows) => {
        //   paginatedRows.value = newPaginatedRows
        // }
    
        // onMounted(() => {
        //   fetchorder()
        function getOptions(options){
  // const optionsArray = options.split("/n")
  let formatOptions = []
  if (options){
    const optionsArray = options.split("\n")
    console.log('options',optionsArray)
    for (let options of optionsArray){
    formatOptions.push({
      label: options,
      value:options
    })
  }
}
return formatOptions
}
function restsetFunction(){
  console.log('Working on clearning or resteing the field_filters',field_filters)
  Object.keys(field_filters).forEach((key) => {
    delete field_filters[key];  
  });
  console.log(field_filters)
  supplier_detail.fetch()
}

  function getComponentType(fieldData){
// console.log("sss",fieldData.fieldtype,fieldData.options)
    const components = {
        Select:Select,
        Color: FormControl,
        Date: DatePicker,
        Link: FormControl,
        TextEditor : FormControl,
        Data: FormControl,
        ReadOnly: 'ReadOnlyComponent'
    }
    return components[fieldData.fieldtype] || FormControl
  }
  function getComponentProps(fieldData){
    const property = {
      Select:{
          options:getOptions(fieldData.options),
          placeholder :fieldData.label,
      },
      Link:{
        size:"sm",
        variant:"subtle",
        placeholder: fieldData.label,
      },
      Date:{
        size:"sm",
        placeholder:fieldData.label,
      },
      Data:{
        size:"sm",
        variant:"subtle",
        placeholder: fieldData.label,
      },
      TextEditor:{
          placeholder:fieldData.label,
          variant :"subtle"
      }
 
    }
    // console.log("property[fieldData.fieldtype]",fieldData.fieldtype,fieldData.fieldname)
    return property[fieldData.fieldtype]
  }
  watch(field_filters, (newFilters) => {
    console.log("new",newFilters)
    // Apply filters and fetch data when filters change
    supplier_detail.fetch(); // Make sure this fetch uses the updated filters
  
    // Remove any empty filters
    for (const key in newFilters) {
      if (newFilters[key] === null || newFilters[key] === '') {
        console.log("deleted ---- the key ")
        delete newFilters[key]; // Remove empty filter keys
      }
    }
  }, { deep: true }); // Use deep watch to monitor nested properties
 
// Fetch initial data
    supplier_detail.fetch(); // })

    const getStatusTheme = (status) => {    
      switch (status) {
        case 'Draft':
          return { theme: "red" };  
        case 'Overdue':
          return { theme: "blue" };
        case 'Cancelled':
          return { theme: "Green" };  
        case 'Return':
          return { theme: "orange" };            
        default:
          return { theme: "gray" };
      }
    }

 
    
        return {
          isSidebarCollapsed,
          columns_data,
          salesdata,
          filter_data,
          field_filters,
          restsetFunction,
          pageLengthCount,
          supplier_detail,
          totalRows,
          // paginatedRows, 
          toggleSidebar,
          OpenClick,
          getComponentType,
          getComponentProps,
          getStatusTheme,
          // updatePaginatedRows,
          order,
         
          supplier_detail
        }
      },
    }
  </script>
    
    <style scoped>
    .head-layout {
      display: flex;
      width: 100%;
      transition: margin-left 0.3s ease;
    }
    .layout {
      display: flex;
      width: 100%;
      height: 80vh;
      transition: margin-left 0.3s ease;
    }
    
    .main-content {
      flex-grow: 1;
      padding: 1rem;
      transition: margin-left 0.3s ease;
      margin-left: 220px; /* Default width of sidebar */
    }
    .head-content {
      flex-grow: 1;
      padding: 0px;
      transition: margin-left 0.3s ease;
      margin-left: 220px; /* Default width of sidebar */
    }
    .collapsed .main-content {
      margin-left: 60px; /* Adjust when sidebar is collapsed */
    }
    .collapsed .head-content {
      margin-left: 60px; /* Adjust when sidebar is collapsed */
    }
    .list-row {
      display: flex;
      justify-content: space-between;
      padding: 10px;
      border-bottom: 1px solid #e5e7eb; /* Gray bottom border */
    }
    
    .row:hover {
      background-color: #f9fafb; /* Light gray background on hover */
    }
    </style>
      