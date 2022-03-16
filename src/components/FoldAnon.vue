<template lang="pug">
#foldAnon.modal.fade(ref="foldAnon" tabindex="-1" data-bs-backdrop="static",
  aria-labelledby="foldAnonLabel" aria-hidden="true")
  .modal-dialog
    .modal-content
      .modal-header
        h5#foldAnonLabel.modal-title User details
      .modal-body
        p Would you like to fold anonymously or choose a name?
      .modal-footer
        button.settings.btn.btn-primary(type="button" @click="setFoldAnon()") Fold Anonymously
        button.settings.btn.btn-primary(type="button" data-bs-toggle="modal" data-bs-target="#userDetails")
          | Choose a name
#userDetails.modal.fade(ref="userDetails" tabindex="-1" data-bs-backdrop="static"
  aria-labelledby="userDetailsLabel" aria-hidden="true")
  .modal-dialog
    .modal-content
      .modal-header
        h5#userDetailsLabel.modal-title User details
      .modal-body
        form
          .row.mb-3
            label.col-sm-3.col-md-3.col-form-label(for="user") Username :
            .col-sm-9
              Field#user.form-control(v-model="config.user" name="user" type="text")
              ErrorMessage.error(name="user")
          .row.mb-3
            label.col-sm-3.col-md-3.col-form-label(for="team") Team Id :
            .col-sm-9
              Field#team.form-control(v-model.number="config.team" name="teamId" type="number" min="0")
              ErrorMessage.error(name="teamId")
          .row.mb-3
            label.col-sm-3.col-md-3.col-form-label(for="passkey") Passkey :
            .col-sm-9
              .input-group
                Field#passkey.form-control(v-model="config.passkey" name="passkey" :type="[ showPasskey ? 'text':'password']")
                button.btn.btn-outline-dark(type="button" @click="showPasskey = !showPasskey")
                  span(v-if="showPasskey")
                    i.fas.fa-eye-slash(aria-hidden="true")
                  span(v-else)
                    i.fas.fa-eye(aria-hidden="true")
              ErrorMessage.error(name="passkey")
      .modal-footer
        button.settings.btn.btn-primary.mr-auto(type="button" data-bs-toggle="modal" data-bs-target="#foldAnon") Go Back
        button.settings.btn.btn-primary(type="button" @click="reset()") Reset
        button.settings.btn.btn-primary(type="button" :disabled="!hasCorrectValues" @click="save()") Save
</template>

<script>
import { onMounted, reactive, toRefs, computed } from 'vue'
import useWebSocket from '../composables/useWebSocket'
import { Modal } from  'bootstrap'
import { useForm, Form, Field, ErrorMessage } from 'vee-validate'
import * as yup from 'yup'

export default {
  name: 'FoldAnon',
  components: { Field, Form, ErrorMessage },
  setup() {
    const { config, send } = useWebSocket()
    const data = reactive({
      foldAnon: null,
      userDetails: null,
      showPasskey: false,
      dummyVal: false,
      config: JSON.parse(JSON.stringify(config.value))
    })

    const schema = yup.object().shape({
      user: yup.string().min(1).max(100).label("Username").matches('^(?![_0-9])[A-Za-z0-9_]+$',
        "Username can contain alphabets, digits and underscores. Username cannot be anonymous, start with a digit or an underscore."),
      teamId: yup.number().integer().min(1).label("Team ID"),
      passkey: yup.string().max(32).label("Passkey").matches('^[0-9A-Fa-f]{32}$',
        "Passkey should be hexadecimal and 32 characters long")
    })

    const { errors } = useForm({validationSchema: schema})

    const reset = () => { data.config = JSON.parse(JSON.stringify(config.value)) }
    const save = () => {
      let msg = { cmd: "config", config: {
        user: cached.config.user,
        team: cached.config.team,
        passkey: cached.config.passkey,
        // fold-anon: true
      }}

      // send(msg);
    }

    const setFoldAnon = () => {
      // Update once foldAnon is available in config
      data.dummyVal = false

      let foldAnonModal = Modal.getOrCreateInstance(data.foldAnon)
      let detailsModal = Modal.getOrCreateInstance(data.userDetails)
      foldAnonModal.hide()
      detailsModal.hide()

      // let msg = { cmd: "config", config: { fold-anon: false }}
      // send(msg)
    }

     const changedData = computed(() => {
      const tmp = JSON.parse(JSON.stringify(data.config));
      const result = Object.entries(tmp).filter(([key]) => {
                      return ((typeof config.value[key] == "undefined") ||
                          (JSON.stringify(config.value[key]) != JSON.stringify(tmp[key])));
        });
      return Object.fromEntries(result);
    })

    const hasCorrectValues = computed(() => {
      !Object.keys(errors.value).length && Object.keys(changedData.value).length
    })

    onMounted(() => {
      let foldAnonModal = Modal.getOrCreateInstance(data.foldAnon)
      if(!data.dummyVal) {
        foldAnonModal.show()
        setTimeout(setFoldAnon(), 15000);
      }
    })

    return { ...toRefs(data), errors , reset, save, setFoldAnon, hasCorrectValues }
  }
}
</script>

<style lang="stylus" scoped>
.error
  color red
  font-size small
</style>