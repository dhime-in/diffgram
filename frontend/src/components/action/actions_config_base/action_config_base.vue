<template>
  <div class="d-flex flex-column" style="height: 100%">
    <action_config_wizard_base
      v-if="display_mode === 'wizard'"
      :action="action"
      :actions_list="actions_list"
      :steps_config="steps_config"
      :action_template="action_template"
      @open_action_selector="$emit('open_action_selector')"
      :project_string_id="project_string_id">
      <template v-slot:triggers>
        <slot name="wizard_triggers"></slot>
      </template>
      <template v-slot:pre_conditions>
        <slot name="wizard_pre_conditions"></slot>
      </template>
      <template v-slot:action_config>
        <slot name="wizard_action_config">
          <p>
            Your Action specific configuration goes here.
          </p>
          <p>
            If you want to override this add the following slot in your action configuration component. It should
            look like this:
          </p>
          <code >
            <pre style="background: dimgrey; color: white">{{sample_code}}</pre>
          </code>
          <p>
            If you want to learn more about developing actions read:
            <a class="secondary--text"

               href="https://dash.readme.com/project/diffgram/v1.0/docs/developing-you-custom-actions">Docs On Developing you Actions</a>
          </p>    action_config_form_base,
        </slot>
      </template>
      <template v-slot:completion_trigger>
        <slot name="wizard_completion_trigger"></slot>
      </template>

    </action_config_wizard_base>


    <action_config_form_base
      v-if="display_mode === 'form'"
      :project_string_id="project_string_id"
      :action="action"
      :actions_list="actions_list"
      :action_template="action_template"
    >
      <template v-slot:triggers>
        <slot name="form_triggers"></slot>
      </template>
      <template v-slot:pre_conditions>
        <slot name="form_pre_conditions"></slot>
      </template>
      <template v-slot:action_config>
        <slot name="form_action_config">
          Your Action specific configuration goes here.
        </slot>
      </template>
      <template v-slot:completion_trigger>
        <slot name="form_completion_trigger"></slot>
      </template>

    </action_config_form_base>

    <div v-if="display_mode === 'ongoing_usage'">
      <slot name="ongoing_usage">
        <h1> YOUR ONGOING USAGE TEMPLATE CODE GOES HERE</h1>
      </slot>

    </div>



  </div>
</template>

<script>
import Action_config_wizard_base from "@/components/action/actions_config_base/action_config_wizard_base";
import action_config_form_base from "@/components/action/actions_config_base/action_config_form_base";
import action_config_mixin from "../action_configurations/action_config_mixin.js";

export default {
  name: "action_config_base",
  mixins: [action_config_mixin],
  components: {
    Action_config_wizard_base,
    action_config_form_base,
  },
  props:{
    steps_config: {
      type: Object,
      required: true
    }
  },
  mounted() {
    this.get_action_template()
  },
  data: function(){
    return{
      job_selected: null,
      show_task_template_wizard: false,
      switch_loading: false,
      sample_code: "<action_config_wizard_base>\n" +
        "   <template v-slot:wizard_action_config>\n" +
        "     Your action configuration code goes here...\n" +
        "   </template>\n" +
        "</action_config_wizard_base>"
    }
  }
}
</script>

<style scoped>

</style>
