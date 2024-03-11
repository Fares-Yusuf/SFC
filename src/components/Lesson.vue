<template>
    <div class="subject-container" v-if="showProduct">
        <div
            class="subject-box"
            v-for="(subject, subjectIndex) in subjects"
            :key="subject.id"
        >
            <figure>
                <img :src="subject.Image" alt="Subject Image" />
            </figure>
            <h2 v-text="subject.title"></h2>
            <p>Location: {{ subject.Location }}</p>
            <p>Price: £{{ subject.Price }}</p>
            <p>Availability: {{ subject.Quantity }}</p>

            <button
                class="button"
                @click="addToCart(subject, subjectIndex)"
                v-if="canAddToCart(subject)"
            >
                Add to cart
            </button>
            <button class="disabled" v-else>Add to cart</button>
        </div>
    </div>
</template>

<script>
export default {
    props: ["subjects", "showProduct"], // Include showProduct as a prop
    methods: {
        addToCart(subject) {
            if (this.canAddToCart(subject)) {
                this.$emit("addToCart", subject);
            }
        },
        canAddToCart(subject) {
            return subject.Quantity > 0;
        },
    },
};
</script>
