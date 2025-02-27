<script setup>
    import { ref, reactive } from 'vue';
    import { useRouter, useRoute } from 'vue-router';
    import { useQuery, useMutation, useQueryClient } from '@tanstack/vue-query';
    import axios from 'axios';
    import { useToast } from 'vue-toastification';
    import VueMultiselect from 'vue-multiselect';
    import Navbar from '../Navbar/Navbar.vue';

    const route = useRoute();
    const router = useRouter();
    const toast = useToast();
    const queryClient = useQueryClient();

    const magasin_id = ref(route.params.magasin_id);
    const productId = ref(route.params.id);

    const form = reactive({
        magasin_id: magasin_id.value,
        code: '',
        designation: '',
        stock_min: '',
        min_sortie: ''
    });

    // Fetch fournisseurs list
    const { data: fournisseurs } = useQuery({
        queryKey: ['fournisseurs'],
        queryFn: async () => {
            const response = await axios.get('http://127.0.0.1:8000/api/fournisseur');
            return response.data;
        },
        onError: () => toast.error('Error loading fournisseurs list'),
    });

    // Fetch product details
    const { data: productData } = useQuery({
        queryKey: ['product', productId.value],
        queryFn: async () => {
            const response = await axios.get(`http://127.0.0.1:8000/api/product/${productId.value}`);
            return response.data.product;
        },
        onSuccess: (newProduct) => {
            if (newProduct) {
                Object.assign(form, {
                    magasin_id: newProduct.magasin_id || magasin_id.value,
                    code: newProduct.code || '',
                    designation: newProduct.designation || '',
                    stock_min: newProduct.stock_min || '',
                    min_sortie: newProduct.min_sortie || ''
                });
            }
        },
        onError: () => toast.error('Failed to fetch product details'),
    });

    // Mutation to update product
    const updateProductMutation = useMutation({
        mutationFn: async (updatedProduct) => {
            await axios.put(`http://127.0.0.1:8000/api/product/${productId.value}`, updatedProduct);
        },
        onSuccess: () => {
            toast.success('Product updated successfully');
            queryClient.invalidateQueries(['product', productId.value]);
            router.push(`/product/show/${magasin_id.value}`);
        },
        onError: () => toast.error('Product was not updated'),
    });

    // Handle Update
    const handleUpdate = () => {
        updateProductMutation.mutate({
            magasin_id: form.magasin_id,
            code: form.code,
            designation: form.designation,
            stock_min: form.stock_min,
            min_sortie: form.min_sortie
        });
    };
</script>


<template>
    <Navbar/>
    <section class="bg-green-50">
        <div class="container m-auto max-w-2xl py-5">
            <div class="bg-white px-6 py-8 mb-4 shadow-md rounded-md border m-4 md:m-0">
                <form @submit.prevent="handleUpdate">
                    <h2 class="text-3xl text-center font-semibold mb-6">Editer le produit</h2>

                    <div class="mb-4">
                        <label class="block text-gray-700 font-bold mb-2">Code</label>
                        <input type="text" v-model="form.code" class="border-gray-300 rounded-md shadow-sm w-full px-4 py-2"/>
                    </div>

                    <div class="mb-4">
                        <label class="block text-gray-700 font-bold mb-2">Designation</label>
                        <input type="text" v-model="form.designation" class="border-gray-300 rounded-md shadow-sm w-full px-4 py-2"/>
                    </div>

                    <div class="mb-4">
                        <label class="block text-gray-700 font-bold mb-2">Stock Minimum</label>
                        <input type="number" step="0.01" v-model="form.stock_min" class="border-gray-300 rounded-md shadow-sm w-full px-4 py-2"/>
                    </div>

                    <div class="mb-4">
                        <label class="block text-gray-700 font-bold mb-2">Minimum Sortie</label>
                        <input type="number" step="0.01" v-model="form.min_sortie" class="border-gray-300 rounded-md shadow-sm w-full px-4 py-2"/>
                    </div>

                    <div>
                        <button type="submit" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded-full w-full">
                            <i class="pi pi-pencil"></i> Editer
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </section>
</template>