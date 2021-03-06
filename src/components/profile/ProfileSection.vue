<template>
  <PanelSection :class="cssClasses" :title="title" :hideContent="editing">
    <template v-slot:meta>
      <a :id="`nav-${section}`" class="profile__anchor"></a>
      <slot name="edit" v-if="editing"></slot>
    </template>
    <template v-slot:header v-if="!editing">
      <EditButton
        class="profile__edit"
        v-if="editable && userOnOwnProfile && !needsLdap"
        :sendTo="{
          name: 'Edit Profile',
          query: {
            section,
          },
        }"
        :section="section"
        :sectionId="section"
      ></EditButton>
    </template>
    <template v-slot:content v-if="!editing">
      <p v-if="empty">{{ emptyMessageText }}</p>
      <slot name="view" v-else></slot>
    </template>
  </PanelSection>
</template>

<script>
import EditButton from '@/components/ui/EditButton.vue';
import EmptyCard from '@/components/profile/view/EmptyCard.vue';
import PanelSection from '@/components/ui/PanelSection.vue';
import { DISPLAY_LEVELS } from '@/assets/js/display-levels';

export default {
  name: 'ProfileSection',
  props: {
    section: String,
    title: String,
    editable: {
      type: Boolean,
      default: true,
    },
    userOnOwnProfile: Boolean,
    empty: {
      type: Boolean,
      default: true,
    },
    message: String,
    messageOwn: String,
    messageNoLdap: String,
    isLdap: Boolean,
    editing: Boolean,
  },
  components: { EditButton, EmptyCard, PanelSection },
  computed: {
    cssClasses() {
      return {
        profile__section: true,
        'profile__section--editing': this.editing,
        'profile__section--disabled': !this.editing && this.empty,
      };
    },
    needsLdap() {
      return this.messageNoLdap && !this.isLdap;
    },
    emptyMessageText() {
      if (
        this.userOnOwnProfile &&
        DISPLAY_LEVELS.private.value ===
          (this.$route.query.pa || DISPLAY_LEVELS.private.value)
      ) {
        if (this.needsLdap) {
          return this.messageNoLdap;
        }
        return this.messageOwn;
      }
      return this.message;
    },
  },
};
</script>

<style>
.profile__section {
  border: 4px solid var(--gray-50);
  background: #fff;
  padding: 1.5em;
  margin: 0 0 2em;
  grid-column: 2 / -1;
  overflow: visible;
  border-radius: var(--cardRadius);
  position: relative;
  max-width: 100%;
}

.profile__section:not(.profile__section--editing):not(.first) {
  padding: calc(2px + 1.5em);
}

.profile__section.first {
  padding: 0;
}

.profile__section.first .profile__section-header {
  margin: 0 0 1.5em 0;
}
@supports (--key: value) {
  .profile__section {
    border: none;
    box-shadow: var(--shadowCard);
  }
}
.profile__section--disabled {
  background-color: var(--gray-20);
  color: var(--gray-50);
  border: 2px solid var(--gray-30);
}
.profile__section.profile__section--editing {
  border: 2px solid var(--blue-60);
}
.profile__section:first-child {
  grid-column: 1 / -1;
}

.profile__section .reporting-structure h3 {
  margin-top: 0;
}

.profile__section .profile__section-header,
.profile__section .panel__section-header {
  padding: 1.5em;
  margin: -1.5em -1.5em 1.5em -1.5em;
  border-bottom: 1px solid var(--gray-30);
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
}

.profile__section:not(.profile__section--editing):not(.first)
  .panel__section-header,
.profile__section:not(.profile__section--editing):not(.first)
  .profile__section-header {
  margin: calc(-1.5em - 2px) calc(-1.5em - 2px) calc(1.5em + 2px)
    calc(-1.5em - 2px);
  padding-top: calc(1.5em + 2px);
  padding-left: calc(1.5em + 2px);
  padding-right: calc(1.5em + 2px);
}
.profile__section .panel__section-header h2,
.profile__section .profile__section-header h2 {
  margin: 0;
  width: 100%;
}

.profile__section .panel__section-content {
  padding: 0;
}
@media (min-width: 35em) {
  .profile__section .panel__section-header h2,
  .profile__section .profile__section-header h2 {
    margin: 0 0 0.75em 0;
  }

  .profile__section .panel__section-header h2,
  .profile__section .profile__section-header h2 {
    margin: 0;
    width: auto;
  }
}
.profile__section--disabled {
  background-color: var(--gray-20);
  color: var(--gray-50);
  border: 2px solid var(--gray-30);
}

.profile__section .edit-button.profile__edit {
  position: absolute;
  top: 2em;
  right: 1.5em;
}
.profile__intro .edit-button {
  position: absolute;
  top: 1em;
  right: 0;
}
</style>
