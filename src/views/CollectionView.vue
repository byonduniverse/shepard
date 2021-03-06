<template>
  <div v-if="currentCollection" class="collection">
    <div>
      <b-button-group class="float-right">
        <b-button
          v-b-modal.create-data-object-modal
          v-b-tooltip.hover
          title="Create"
          variant="primary"
        >
          <CreateIcon />
        </b-button>
        <b-button
          v-b-modal.edit-collection-modal
          v-b-tooltip.hover
          title="Edit"
          variant="light"
        >
          <EditIcon />
        </b-button>
        <b-button
          v-b-modal.permissions-modal
          v-b-tooltip.hover
          title="Edit Permissions"
          variant="light"
        >
          <PermissionsIcon />
        </b-button>
        <b-button
          v-b-modal.delete-confirmation-modal
          v-b-tooltip.hover
          title="Delete"
          variant="dark"
        >
          <DeleteIcon />
        </b-button>
      </b-button-group>
      <h3>{{ currentCollection.name }}</h3>
    </div>
    <p>
      Collection ID: {{ currentCollection.id }}
      <CreatedByLine
        :created-at="currentCollection.createdAt"
        :created-by="currentCollection.createdBy"
        tooltip
      />
      <CreatedByLine
        v-if="currentCollection.updatedBy"
        :created-at="currentCollection.updatedAt"
        :created-by="currentCollection.updatedBy"
        updated
        tooltip
      />
    </p>

    <GenericDescription
      v-if="currentCollection.description"
      :text="currentCollection.description"
    />

    <GenericCollapse v-if="attributeItems.length" title="Attributes" visible>
      <b-table striped small :items="attributeItems"> </b-table>
    </GenericCollapse>

    <GenericCollapse
      v-if="currentCollection.dataObjectIds.length"
      title="Data Objects"
      visible
    >
      <ChildrenList
        :current-collection-id="currentCollection.id"
        :parent-id="-1"
      />
    </GenericCollapse>

    <CollectionModal
      :current-collection="currentCollection"
      modal-id="edit-collection-modal"
      modal-name="Edit Collection"
      @collectionChanged="retrieveCollection()"
    />
    <DataObjectModal
      :current-collection-id="currentCollectionId"
      modal-id="create-data-object-modal"
      modal-name="Create Data Object"
    />
    <DeleteConfirmationModal
      modal-id="delete-confirmation-modal"
      modal-name="Confirm to delete collection"
      :modal-text="
        'Do you really want do delete the collection with name ' +
        currentCollection.name +
        '?'
      "
      @confirmation="handleDelete()"
    />
    <PermissionsModal
      modal-id="permissions-modal"
      modal-name="Edit Permissions"
      :entity-id="currentCollectionId"
      :old-permissions="permissions"
      @update="updatePermissions($event)"
    />
  </div>
</template>

<script lang="ts">
import Vue, { VueConstructor } from "vue";
import { Collection, Permissions } from "@dlr-shepard/shepard-client";
import { CollectionVue } from "@/utils/api-mixin";
import ChildrenList from "@/components/dataobjects/ChildrenList.vue";
import CollectionModal from "@/components/dataobjects/CollectionModal.vue";
import DataObjectModal from "@/components/dataobjects/DataObjectModal.vue";
import GenericCollapse from "@/components/generic/GenericCollapse.vue";
import GenericDescription from "@/components/generic/GenericDescription.vue";
import CreatedByLine from "@/components/generic/CreatedByLine.vue";
import DeleteConfirmationModal from "@/components/DeleteConfirmationModal.vue";
import PermissionsModal from "@/components/PermissionsModal.vue";

interface CollectionData {
  currentCollection?: Collection;
  permissions?: Permissions;
  attributeItems: Array<{ key: string; value: string }>;
}

export default (
  Vue as VueConstructor<Vue & InstanceType<typeof CollectionVue>>
).extend({
  components: {
    GenericCollapse,
    GenericDescription,
    CreatedByLine,
    ChildrenList,
    DataObjectModal,
    CollectionModal,
    DeleteConfirmationModal,
    PermissionsModal,
  },
  mixins: [CollectionVue],
  data() {
    return {
      currentCollection: undefined,
      permissions: undefined,
      attributeItems: [],
    } as CollectionData;
  },
  computed: {
    currentCollectionId(): number {
      return Number(this.$router.currentRoute.params.collectionId);
    },
  },
  mounted() {
    this.retrieveCollection();
    this.retrievePermissions();
  },
  methods: {
    retrieveCollection() {
      this.collectionApi
        ?.getCollection({ collectionId: this.currentCollectionId })
        .then(response => {
          this.currentCollection = response;
          this.attributeItems = [];
          if (this.currentCollection?.attributes !== undefined) {
            Object.entries(this.currentCollection.attributes).forEach(
              ([key, value]) =>
                this.attributeItems.push({ key: key, value: value }),
            );
          }
        })
        .catch(e => {
          console.log("Error while fetching collection: " + e.statusText);
        })
        .finally();
    },
    handleDelete() {
      this.collectionApi
        ?.deleteCollection({ collectionId: this.currentCollectionId })
        .then(() => {
          this.$router.push({ name: "Explore" });
        })
        .catch(e => {
          console.log("Error while deleting collection: " + e.statusText);
        })
        .finally();
    },
    retrievePermissions() {
      this.collectionApi
        ?.getCollectionPermissions({ collectionId: this.currentCollectionId })
        .then(response => {
          this.permissions = response;
        })
        .catch(e => {
          console.log("Error while fetching permissons: " + e.statusText);
        })
        .finally();
    },
    updatePermissions(perms: Permissions) {
      this.collectionApi
        ?.editCollectionPermissions({
          collectionId: this.currentCollectionId,
          permissions: perms,
        })
        .then(response => {
          this.permissions = response;
        })
        .catch(e => {
          console.log("Error while edit permissons: " + e.statusText);
        })
        .finally();
    },
  },
});
</script>

<style scoped>
.listElement {
  border: solid thin;
  border-color: #e3e3e3;
  border-radius: 5px;
  padding: 20px;
}
</style>
