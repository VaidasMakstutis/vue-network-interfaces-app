<template>
  <div>
    <a-modal
      :title="'Interface ' + interfaceName"
      v-model="showModalVisible"
      :width="500"
      @cancel="handleClose"
    >
      <vuci-form uci-config="task1" @applied="$emit('onSubmit')">
        <vuci-named-section
          type="interface"
          :name="sid"
          :label="'Network interface'"
          :collapsible="false"
          v-slot="{ s }"
          :card="false"
        >
          <vuci-form-item-select
            :uci-section="s"
            :label="'Protocol'"
            name="protocol"
            :options="protocols"
            @change="handleChange"
            :required="true"
          />
          <vuci-form-item-input
            :uci-section="s"
            :label="'Address'"
            name="address"
            depend="protocol == 'static'"
            rules="ip4addr"
            :required="true"
          />
          <vuci-form-item-input
            :uci-section="s"
            :label="'Netmask'"
            name="netmask"
            depend="protocol == 'static'"
            rules="netmask4"
            :required="true"
          />
          <vuci-form-item-input
            :uci-section="s"
            :label="'Gateway'"
            name="gateway"
            depend="protocol == 'static'"
            rules="ip4addr"
            :required="true"
          />
          <vuci-form-item-list
            :uci-section="s"
            :label="'DNS'"
            name="dns"
            depend="protocol == 'static'"
            rules="ip4addr"
          />
        </vuci-named-section>
      </vuci-form>
      <template #footer>
        <div></div>
      </template>
    </a-modal>
  </div>
</template>

<script>
export default {
  props: { sid: String, interfaceName: String, interfaceModal: Boolean },
  data () {
    return {
      showModalVisible: this.interfaceModal,
      protocols: [
        ['static', 'Static address'],
        ['dhcp', 'DHCP Client']
      ]
    }
  },
  methods: {
    handleChange (self) {
      if (self.model === 'dhcp') {
        this.$uci.set('task1', this.sid, 'dns', undefined)
        this.$uci.set('task1', this.sid, 'netmask', undefined)
        this.$uci.set('task1', this.sid, 'gateway', undefined)
        this.$uci.set('task1', this.sid, 'address', undefined)
        this.$uci.set('task1', this.sid, 'workaround', undefined)
      } else {
        this.$uci.reset()
      }
    },
    handleClose () {
      this.$emit('onSubmit', this.showModalVisible = false)
    }
  }
}
</script>

<style>
</style>
