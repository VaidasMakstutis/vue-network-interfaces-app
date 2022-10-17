<template>
  <div>
    <vuci-form uci-config="task1" :key="reRenderTable">
      <vuci-typed-section type="interface" :columns="columns">
        <template #name="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="name" />
        </template>
        <template #address="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="address" />
        </template>
        <template #netmask="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="netmask" />
        </template>
        <template #buttons="{ s }">
          <a-button
            type="primary"
            @click="handleEditInterface(s['.name'], s.name, s.protocol)"
            v-text="'Edit'"
            style="margin-right: 10px"
          ></a-button>
          <a-button
            type="danger"
            @click="showDelete(s['.name'], s.name)"
            v-text="'Delete'"
          ></a-button>
        </template>
      </vuci-typed-section>
      <template #footer>
        <div style="margin: 20px">
          <p class="errors" v-if="errors.length" style="color: rgb(255,0,0); padding-bottom: 20px">
                <span v-for="error in errors" :key="error">{{ error }}</span>
          </p>
          <div style="">
            <label style="margin-top: 5px">Interface name:</label>
            <a-input
              @pressEnter="validateInterfaceName()"
              v-model="newItemName"
              style="width: 200px; margin: 0 15px"
            ></a-input>
            <a-button
              @click="validateInterfaceName()"
              type="primary"
              v-text="'Create'"
            ></a-button>
          </div>
        </div>
      </template>
    </vuci-form>
    <interface-modal
      :key="reRenderModal"
      :sid="currentSid"
      :interfaceName="selectedItemName"
      :interfaceModal="showModal"
      @onSubmit="handleSubmit"
    ></interface-modal>
  </div>
</template>

<script>
import InterfaceModal from './components/interfaceModal.vue'

export default {
  components: { InterfaceModal },
  data () {
    return {
      showModal: false,
      newItemName: '',
      available: true,
      selectedItemName: '',
      currentSid: '',
      reRenderTable: 1,
      reRenderModal: false,
      columns: [
        { name: 'name', label: 'Interface name' },
        { name: 'address', label: 'Address' },
        { name: 'netmask', label: 'Netmask' },
        { name: 'buttons', label: 'Actions' }
      ],
      errors: []
    }
  },
  methods: {
    validateInterfaceName () {
      this.errors = []
      const sections = this.$uci.sections('task1')
      sections.forEach((section) => {
        if (this.newItemName === section.name) {
          this.available = false
        }
      })

      if (this.available === false) {
        this.errors.push(
          `Interface with name ${this.newItemName} is already exist!`
        )
        this.available = true
      } else if (!this.newItemName.length) {
        this.errors.push('Please enter interface name!')
      } else {
        this.handleAddInterface()
      }
    },
    async handleAddInterface () {
      this.$spin()
      const sid = this.$uci.add('task1', 'interface')
      await this.$uci.set('task1', sid, 'name', this.newItemName)
      await this.$uci.set('task1', sid, 'protocol', 'static')
      await this.$uci.save()
      this.findLatestSid().then((res) => {
        this.currentSid = res
        this.selectedItemName = this.newItemName
        this.newItemName = ''
        this.handleModalRefresh()
        this.showModal = true
      })
      this.$spin(false)
    },
    async findLatestSid () {
      await this.$uci.load('task1')
      const data = this.$uci.sections('task1')
      return data[data.length - 1]['.name']
    },
    handleEditInterface (sid, name, protocol) {
      this.selectedItemName = name
      this.currentSid = sid
      this.handleModalRefresh()
      this.selectedProtocol = protocol
      this.showModal = true
    },
    async handleDeleteInterface (sid) {
      this.$spin()
      await this.$uci.del('task1', sid)
      await this.$uci.save()
      await this.$uci.apply('task1')
      this.handleTableRefresh()
      this.$spin(false)
    },
    showDelete (sid, interfaceName) {
      this.$confirm({
        content: `Are you sure that you want to delete interface with name ${interfaceName}?`,
        onOk: () => {
          this.handleDeleteInterface(sid)
        }
      })
    },
    handleSubmit (option) {
      this.showModal = option
      this.handleTableRefresh()
      this.handleModalRefresh()
    },
    handleTableRefresh () {
      this.reRenderTable += 1
    },
    handleModalRefresh () {
      this.reRenderModal = !this.reRenderModal
    }
  }
}
</script>

<style>
</style>
