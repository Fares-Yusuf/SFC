<template>
    <div class="Cart-container">
        <h3>Welcome to the checkout page</h3>
        <div class="Cart-page">
            <div class="cart-box" v-for="(item, key) in cart" :key="key">
                <figure>
                    <img :src="item.Image" alt="Subject Image" />
                </figure>
                <h2>{{ item.title }}</h2>
                <p>Price: £{{ item.Price }}</p>
                <p>Quantity: {{ item.Quantity }}</p>
                <button class="button" @click="removeCartItem(item)">
                    Remove
                </button>
            </div>
        </div>
        <hr />
    </div>
</template>

<script>
export default {
    props: ["cart"],
    methods: {
        removeCartItem: function (item) {
            this.$emit("removeCartItem", item);
        },
    },
};
</script>
