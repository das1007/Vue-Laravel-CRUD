<template>
    <div>
        <b-button
            v-b-modal.addUserModel
            @click="$bvModal.show('addUserModel')"
            class="addButton"
            >Create New Product
        </b-button>
        <b-table striped hover :items="products" :fields="fields">
            <template v-slot:cell(actions)="{ item: { id } }">
                <b-dropdown variant="primary" text="Actions">
                    <b-dropdown-item
                        v-b-modal.addUserModel
                        v-on:click="editProduct(id)"
                        >Edit</b-dropdown-item
                    >
                    <b-dropdown-item v-on:click="deleteProduct(id)"
                        >Delete</b-dropdown-item
                    >
                </b-dropdown>
            </template>
        </b-table>
        <b-modal
            id="addUserModel"
            size="lg"
            title="Add Policy"
            body-class="bg-white"
            cancel-variant="default"
            class="addUserModelClose"
            :hide-footer="true"
        >
            <b-form ref="form">
                <input name="id" type="hidden" v-model="this.editData.id" />
                <b-form-group id="name" label="Enter Name:" label-for="name">
                    <b-form-input
                        id="name"
                        name="name"
                        type="text"
                        placeholder="Enter Name"
                        v-model="this.editData.name"
                        required
                    ></b-form-input>
                </b-form-group>

                <b-form-group
                    id="detail"
                    label="Your Detail:"
                    label-for="detail"
                >
                    <b-form-input
                        id="detail"
                        type="text"
                        name="detail"
                        placeholder="Enter Detail"
                        v-model="this.editData.detail"
                        required
                    ></b-form-input>
                </b-form-group>
                <b-button
                    v-if="this.setEdit == true"
                    type="submit"
                    variant="primary"
                    @click="onSubmit"
                    >Update</b-button
                >
                <b-button
                    v-else
                    type="submit"
                    @click="onSubmit"
                    variant="primary"
                    >Submit</b-button
                >
                <b-button type="reset" variant="danger">Reset</b-button>
            </b-form>
        </b-modal>
    </div>
</template>

<script>
export default {
    data() {
        return {
            fields: ["name", "detail", "actions"],
            products: [],
            name: "",
            detail: "",
            editData: "",
            setEdit: false
        };
    },
    mounted() {
        this.getAllProduct();
    },
    methods: {
        getAllProduct: function() {
            this.axios
                .get("http://localhost:8000/api/products/")
                .then(response => {
                    return (this.products = response.data.map(item => {
                        return {
                            id: item.id,
                            name: item.name,
                            detail: item.detail
                        };
                    }));
                });
        },
        deleteProduct(id) {
            this.$swal({
                title: "Are you sure?",
                text: "You can't revert your action",
                type: "warning",
                icon: "warning",
                showCancelButton: true,
                confirmButtonText: "Yes Delete it!",
                cancelButtonText: "No, Keep it!",
                showCloseButton: true,
                showLoaderOnConfirm: true
            }).then(result => {
                if (result.value) {
                    this.axios
                        .delete(`http://localhost:8000/api/products/${id}`)
                        .then(response => {
                            this.$swal(
                                "Deleted",
                                "You successfully deleted this file",
                                "success"
                            );
                            this.getAllProduct();
                        });
                } else {
                    this.$swal(
                        "Cancelled",
                        "Your file is still intact",
                        "info"
                    );
                }
            });
        },
        editProduct(id) {
            this.setEdit = true;
            this.axios
                .get(`http://localhost:8000/api/products/${id}`)
                .then(response => {
                    this.editData = response.data;
                    // this.editName(this.editData);
                });
        },
        onSubmit(e) {
            e.preventDefault();
            if (this.setEdit == false) {
                var data = {
                    name: this.$refs.form.name.value,
                    detail: this.$refs.form.detail.value
                };
                this.axios
                    .post("http://localhost:8000/api/products", data)
                    .then(
                        response => (
                            this.$bvModal.hide("addUserModel"),
                            this.$swal(
                                "Inserted",
                                "You successfully Inserted record.",
                                "success"
                            ),
                            this.getAllProduct(response)
                        )
                    );
            }

            if (this.setEdit == true) {
                var id = this.$refs.form.id.value;
                var data = {
                    name: this.$refs.form.name.value,
                    detail: this.$refs.form.detail.value
                };
                this.axios
                    .put(`http://localhost:8000/api/products/${id}`, data)
                    .then(
                        response => (
                            this.$bvModal.hide("addUserModel"),
                            this.$swal(
                                "Updated",
                                "You successfully Updated record.",
                                "success"
                            ),
                            this.getAllProduct(response),
                            this.setEdit = false,
                            this.$refs["name"].value = ""
                        )
                    );
            }
        }
    }
};
</script>
<style>
button.btn.addButton.btn-secondary {
    margin-left: 85%;
    margin-bottom: 1%;
}
.modal-backdrop {
    background-color: unset !important;
}
</style>
