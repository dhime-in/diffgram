<template>
    <v-toolbar
        dense
        width="100%"
        elevation="0"
        fixed
        :height="height"
        style="overflow: hidden; padding: 0; border-bottom: 1px solid #e0e0e0; border-top: 1px solid #e0e0e0"
    >
        <v-toolbar-items v-if="task">
            <div v-show="task && task.id">
                <v-layout>
                <div style="width: 10px" />
                <ui_schema name="logo">
                    <ahref_seo_optimal
                    v-if="$store.getters.get_ui_schema('logo', 'visible')"
                    :href="'/me'"
                    >
                    <div class="pt-2 pr-3 clickable">
                        <img
                        src="https://storage.googleapis.com/diffgram-002/public/logo/diffgram_logo_word_only.png"
                        height="30px"
                        />
                    </div>
                    </ahref_seo_optimal>
                </ui_schema>

                <tooltip_button
                    ui_schema_name="home"
                    color="primary"
                    datacy="toolbar_home_button"
                    :icon_style="true"
                    icon="mdi-home"
                    tooltip_message="Home"
                    @click="$router.push('/me')"
                    :bottom="true"
                >
                </tooltip_button>

                <tooltip_button
                    ui_schema_name="task_list"
                    color="primary"
                    :icon_style="true"
                    icon="mdi-playlist-play"
                    datacy="go-to-task-list"
                    tooltip_message="Task List"
                    @click="$router.push('/job/' + task.job_id)"
                    :bottom="true"
                >
                </tooltip_button>

                <v-divider vertical></v-divider>

                <tooltip_button
                    color="primary"
                    icon="mdi-undo"
                    tooltip_message="Undo (ctrl+z)"
                    ui_schema_name="undo"
                    :disabled="undo_disabled"
                    :icon_style="true"
                    :bottom="true"
                    @click="$emit('undo')"
                />

                <tooltip_button
                    color="primary"
                    icon="mdi-redo"
                    tooltip_message="Redo (ctrl+y)"
                    ui_schema_name="redo"
                    :disabled="redo_disabled"
                    :icon_style="true"
                    :bottom="true"
                    @click="$emit('redo')"
                />

                <v-divider vertical v-if="task && task.status !== 'complete'" ></v-divider>

                <v_is_complete
                    v-if="task && task.status !== 'complete'"
                    :project_string_id="project_string_id"
                    :current_file="file ? file : task.file"
                    :task="task"
                    @complete_task="$emit('complete_task')"
                    @replace_file="$emit('replace_file', $event)"
                    @on_next="$emit('change_task', 'next')"
                    @on_task_annotation_complete_and_save="
                    $emit('on_task_annotation_complete_and_save')
                    "
                    :save_and_complete="true"
                    :loading="save_loading"
                    :disabled="save_loading || (!file && !task)"
                    :task_id="task ? task.id : undefined"
                >
                </v_is_complete>

                <div>
                    <tooltip_button
                        v-if="task && task.id && task.status == 'available'"
                        ui_schema_name="defer"
                        @click="$emit('task_update_toggle_deferred')"
                        :loading="save_loading"
                        :disabled="
                            save_loading ||
                            (file == undefined && task == undefined)
                        "
                        color="primary"
                        :icon_style="true"
                        icon="mdi-debug-step-over"
                        tooltip_message="Defer"
                        :bottom="true"
                    />
                </div>

                <v-divider vertical></v-divider>

                <div style="width: 310px; overflow-y: hidden; max-height: 50px">
                    <div class="pl-2 pr-3 pt-4">
                        <label_select_annotation
                            :project_string_id="project_string_id"
                            :label_file_list="label_list"
                            :label_file_colour_map="label_file_colour_map"
                            :loading="loading"
                            :request_refresh_from_project="true"
                            :show_visibility_toggle="true"
                            @change="$emit('change_label_file', $event)"
                            @update_label_file_visible="$emit('change_label_visibility', $event)"
                        />
                    </div>
                </div>

                <v-divider vertical></v-divider>
                
                <div class="pl-3 pr-3 pt-4" style="width: 200px; overflow-y: hidden; max-height: 50px">
            <!-- instance_selector -->
                    <diffgram_select
                        :item_list="instance_type_list"
                        data_cy="instance-type-select"
                        v-model="instance_type"
                        label="New Instance Type"
                        :disabled="loading || !draw_mode"
                        @change="$emit('change_instance_type', instance_type)"
                    >
                    </diffgram_select>
                </div>

                <v-divider vertical></v-divider>

                <div style="overflow-y: hidden; max-height: 50px">
                    <div class="pl-3 pt-3 pr-2">
                        <v-switch
                            :label_file="mode_text"
                            data-cy="edit_toggle"
                            :input-value="draw_mode"
                            @change="$emit('edit_mode_toggle')"
                            :label="mode_text"
                        >
                        </v-switch>
                    </div>
                </div>

                <v-divider vertical></v-divider>

                <div>
                    <tooltip_button
                        ui_schema_name="save"
                        @click="$emit('save')"
                        datacy="save_button"
                        :loading="save_loading"
                        :disabled="
                            !has_changed ||
                            save_loading
                        "
                        color="primary"
                        icon="save"
                        tooltip_message="Save Image / Frame"
                        :icon_style="true"
                        :bottom="true"
                        >
                    </tooltip_button>
                </div>
                <div class="has-changed">
                    <div style="width: 100px">
                    <span v-if="save_loading"> Saving </span>
                    <span v-else>
                        <span v-if="has_changed">Changes Detected...</span>
                        <span v-else>Saved</span>
                    </span>
                    </div>
                </div>

                <v-divider vertical></v-divider>

                <div>
                    <task_status 
                        v-if="task && task.id && task.job"
                        :task_status="task.status"
                        :task_comment="task.task_comment"
                        :allow_reviews="task.job.allow_reviews"
                    />
                </div>

                <v-divider vertical></v-divider>

                <div>
                    <tooltip_button
                        tooltip_message="Previous File"
                        @click="$emit('change_task', 'previous')"
                        color="primary"
                        icon="mdi-chevron-left-circle"
                        :icon_style="true"
                        :bottom="true"
                        :disabled="loading || save_loading"
                    >
                    </tooltip_button>
                    <!-- TODO Move some of disabled logic into functions don't like having
                        so much of it here as it gets more complext -->
                </div>
                <div>
                    <tooltip_button
                        tooltip_message="Next File"
                        @click="$emit('change_task', 'next')"
                        color="primary"
                        icon="mdi-chevron-right-circle"
                        :icon_style="true"
                        :bottom="true"
                        :disabled="loading || save_loading"
                    >
                    </tooltip_button>
                </div>

                <v-divider vertical></v-divider>

                <button_with_menu
                    tooltip_message="Hotkeys"
                    color="primary"
                    icon="mdi-keyboard-settings"
                    :close_by_button="true"
                >
                    <template slot="content">
                        <geo_hotkeys />
                    </template>
                </button_with_menu>

                <v_annotation_trainer_menu
                    v-if="task && task.id"
                    :job_id="task.job_id"
                    :task="task"
                >
                </v_annotation_trainer_menu>

                <v-divider vertical></v-divider>
                </v-layout>
            </div>
        </v-toolbar-items>
        <v-toolbar-items v-else>
            <div style="width: 10px" />
            <tooltip_button
                color="primary"
                icon="mdi-undo"
                tooltip_message="Undo (ctrl+z)"
                ui_schema_name="undo"
                :disabled="undo_disabled"
                :icon_style="true"
                :bottom="true"
                @click="$emit('undo')"
            />

            <tooltip_button
                color="primary"
                icon="mdi-redo"
                tooltip_message="Redo (ctrl+y)"
                ui_schema_name="redo"
                :disabled="redo_disabled"
                :icon_style="true"
                :bottom="true"
                @click="$emit('redo')"
            />
            
            <v-divider vertical></v-divider>

            <div class="pl-2 pr-3 pt-4">
                <label_schema_selector
                :project_string_id="project_string_id"
                :initial_schema="label_schema"
                @change="$emit('change_label_schema', $event)"
                @update_label_file_visible="$emit('update_label_file_visibility', $event)"
                >
                </label_schema_selector>
            </div>
            
            <v-divider vertical></v-divider>

            <div style="width: 310px; overflow-y: hidden; max-height: 50px">
                <div class="pl-2 pr-3 pt-4">
                <label_select_annotation
                    :project_string_id="project_string_id"
                    :label_file_list="label_list"
                    :schema_id="label_schema.id"
                    :label_file_colour_map="label_file_colour_map"
                    :loading="loading"
                    :request_refresh_from_project="true"
                    :show_visibility_toggle="true"
                    @change="$emit('change_label_file', $event)"
                    @update_label_file_visible="$emit('change_label_visibility', $event)"
                />
                </div>
            </div>

            <v-divider vertical></v-divider>
            
            <div class="pl-3 pr-3 pt-4" style="max-width: 200px">
          <!-- instance_selector -->
                <diffgram_select
                    :item_list="instance_type_list"
                    data_cy="instance-type-select"
                    v-model="instance_type"
                    label="New Instance Type"
                    :disabled="loading || !draw_mode"
                    @change="$emit('change_instance_type', instance_type)"
                >
                </diffgram_select>
            </div>

            <v-divider vertical></v-divider>

            <div class="pl-3 pt-3 pr-2">
                <v-switch
                    :label_file="mode_text"
                    data-cy="edit_toggle"
                    :input-value="draw_mode"
                    @change="$emit('edit_mode_toggle')"
                    :label="mode_text"
                >
                </v-switch>
            </div>

            <v-divider vertical></v-divider>

            <div>
                <tooltip_button
                    ui_schema_name="save"
                    @click="$emit('save')"
                    datacy="save_button"
                    :loading="save_loading"
                    :disabled="
                        !has_changed ||
                        save_loading
                    "
                    color="primary"
                    icon="save"
                    tooltip_message="Save Image / Frame"
                    :icon_style="true"
                    :bottom="true"
                    >
                </tooltip_button>
            </div>
            <div class="has-changed">
                <div style="width: 100px">
                <span v-if="save_loading"> Saving </span>
                <span v-else>
                    <span v-if="has_changed">Changes Detected...</span>
                    <span v-else>Saved</span>
                </span>
                </div>
            </div>

            <v-divider vertical></v-divider>

            <div>
                <tooltip_button
                    tooltip_message="Previous File"
                    @click="$emit('change_file', 'previous')"
                    color="primary"
                    icon="mdi-chevron-left-circle"
                    :icon_style="true"
                    :bottom="true"
                    :disabled="loading || save_loading"
                >
                </tooltip_button>
                <!-- TODO Move some of disabled logic into functions don't like having
                    so much of it here as it gets more complext -->
            </div>
            <div>
                <tooltip_button
                    tooltip_message="Next File"
                    @click="$emit('change_file', 'next')"
                    color="primary"
                    icon="mdi-chevron-right-circle"
                    :icon_style="true"
                    :bottom="true"
                    :disabled="loading || save_loading"
                >
                </tooltip_button>
            </div>

            <v-divider vertical></v-divider>

            <button_with_menu
                tooltip_message="Hotkeys"
                color="primary"
                icon="mdi-keyboard-settings"
                :close_by_button="true"
            >
                <template slot="content">
                    <geo_hotkeys />
                </template>
            </button_with_menu>

            <v-divider vertical></v-divider>

        </v-toolbar-items>
    </v-toolbar>
</template>

<script>
import Vue from 'vue'
import label_select_annotation from "../label/label_select_annotation.vue"
import label_schema_selector from "../label/label_schema_selector.vue"
import task_status from "../annotation/task_status.vue"
import geo_hotkeys from "./geo_hotkeys.vue"

export default Vue.extend({
    name: "geo_toolbar",
    components: {
        label_select_annotation,
        geo_hotkeys,
        label_schema_selector,
        task_status
    },
    props: {
        undo_disabled: {
            type: Boolean,
            required: true
        },
        redo_disabled: {
            type: Boolean,
            required: true
        },
        has_changed: {
            type: Boolean,
            default: false
        },
        save_loading: {
            type: Boolean,
            default: false
        },
        height: {
            type: String,
            default: '50px'
        },
        project_string_id: {
            type: String,
            required: true
        },
        label_list: {
            type: Array,
            required: true
        },
        loading: {
            type: Boolean,
            default: false
        },
        label_file_colour_map: {
            type: Object,
            requered: true
        },
        task: {
            type: Object,
            default: undefined
        },
        file: {
            type: Object,
            default: undefined
        },
        instance_type_list: {
            type: Array,
            default: []
        },
        draw_mode: {
            type: Boolean,
            default: {}
        },
        label_schema: {
            type: Object,
            required: true
        }
    },
    data() {
        return {
            instance_type: "geo_circle",
        }
    },
    computed: {
        mode_text: function () {
            if (this.draw_mode == true) {
                return "Draw";
            } else {
                return "Edit";
            }
        }, 
    }
})
</script>