<script setup>
import Checkout from "./components/Checkout.vue";
import Lesson from "./components/Lesson.vue";
</script>
<script>
export default {
    components: {
        Checkout,
        Lesson,
    },
    data() {
        return {
            url: "https://afterschoollessons-env.eba-46im9ecw.eu-west-2.elasticbeanstalk.com/lessons",
            viewTestConsole: false,
            sitename: "After School Club",
            showProduct: true, // default
            sortCriteria: "Title_asc", // Default sorting criteria
            subjects: [], // JSON data stored in an array
            cart: [], // array to store items in shopping cart
            sortPrice: "Price", // Default sorting type - Price
            sortTitle: "Title", // Default sorting type - title
            sortLocation: "Location", // Default sorting type - location
            sortQuantity: "Quantity", // Default sorting type - Quantity
            sortCategory: null, // Default selection
            sortOrder: null, // Default selection
            orderSubmitted: false, // Flag to indicate if the order has been submitted
            name: "", // Validation for checkout
            phone: "", // Validation for checkout
            isCheckoutEnabled: false, // Validation for checkout
            searchKeyword: "", // Add a new data property for the search keyword
            searchResults: [], // Add a new data property for search results
            showCheckout: false, // Flag to control the visibility of the checkout section
        };
    },
    mounted: function () {
        // Fetch data as soon as the homepage loads
        this.fetchSubjects();
    },

    methods: {
        fetchSubjects() {
            // Fetch initial subjects data when the component is created
            fetch(
                "https://afterschoollessons-env.eba-46im9ecw.eu-west-2.elasticbeanstalk.com/lessons"
            )
                .then((response) => response.json())
                .then((data) => {
                    console.log("Data from MongoDB:", data);
                    // Use the data in your front-end application
                    this.subjects = data;
                })
                .catch((error) => console.error("Error fetching data:", error))
                .finally(() => {
                    // Once the data is loaded, you can set showProduct to true
                    this.showProduct = true;
                });
        },

        // method to add subjects to cart
        addToCart: function (subject) {
            let itemIndex = this.cart.findIndex((ct) => ct.id === subject.id);
            if (itemIndex === -1) {
                this.cart.push({
                    ...subject,
                    Quantity: 1,
                });
            } else {
                this.cart[itemIndex].Quantity++;
            }

            // Finds the subject in the subjects array (not filteredSubjects) and update its Quantity
            const subjectIndex = this.subjects.findIndex(
                (s) => s.id === subject.id
            );
            if (subjectIndex !== -1) {
                this.subjects[subjectIndex].Quantity--;
            }
        },

        // Function to handle sorting based on the selected category and order
        sortSubject: function () {
            [this.sortCategory, this.sortOrder] = this.sortCriteria.split("_");
            if (this.sortCategory && this.sortOrder) {
                if (this.sortCategory === "Title") {
                    // Sort subjects by title
                    this.subjects.sort((a, b) => {
                        if (this.sortOrder === "asc") {
                            return a.title.localeCompare(b.title);
                        } else {
                            return b.title.localeCompare(a.title);
                        }
                    });
                } else if (this.sortCategory === "Location") {
                    // Sort subjects by location.
                    this.subjects.sort((a, b) => {
                        if (this.sortOrder === "asc") {
                            return a.Location.localeCompare(b.Location);
                        } else {
                            return b.Location.localeCompare(a.Location);
                        }
                    });
                } else if (this.sortCategory === "Quantity") {
                    // Sort subjects by Quantity
                    if (this.sortOrder === "asc") {
                        this.subjects.sort((a, b) => a.Quantity - b.Quantity);
                    } else {
                        this.subjects.sort((a, b) => b.Quantity - a.Quantity);
                    }
                } else if (this.sortCategory === "Price") {
                    // Sort subjects by Price
                    if (this.sortOrder === "asc") {
                        this.subjects.sort((a, b) => a.Price - b.Price);
                    } else {
                        this.subjects.sort((a, b) => b.Price - a.Price);
                    }
                }
            }
        },

        // Method to show Cart page
        showCart() {
            if (this.cartItemCount == 0) {
                this.showProduct = true;
                this.showCheckout = false; // Hide the checkout section when the cart is empty
            } else {
                this.showProduct = !this.showProduct;
                this.showCheckout = !this.showCheckout; // Toggle the visibility of the checkout section
                this.viewTestConsole = false;
            }
        },

        toggleShowProduct() {
            // Reset the sorting options to default
            this.sortCategory = null;
            this.sortOrder = null;

            // Fetch the original data from MongoDB
            fetch(
                "https://afterschoollessons-env.eba-46im9ecw.eu-west-2.elasticbeanstalk.com/lessons"
            )
                .then((response) => response.json())
                .then((data) => {
                    console.log("Data from MongoDB:", data);
                    // Use the data in your front-end application
                    this.subjects = data;
                })
                .catch((error) => console.error("Error fetching data:", error));
        },

        // Remove Cart items
        removeCartItem: function (item) {
            if (item.Quantity > 1) {
                item.Quantity--;
            } else {
                // Remove the item from the cart when the Quantity reaches zero
                this.cart.splice(this.cart.indexOf(item), 1);
            }

            // Find the subject in the subjects array and increase its Quantity
            const subjectIndex = this.subjects.findIndex(
                (subject) => subject.id === item.id
            );
            if (subjectIndex !== -1) {
                this.$set(
                    this.subjects[subjectIndex],
                    "Quantity",
                    this.subjects[subjectIndex].Quantity + 1
                );
            }

            // Check if the cart is empty
            if (this.cart.length === 0) {
                // Navigate to the lessons
                this.showProduct = true;
                this.showCheckout = false;
                // Update the subjects after modifying the cart
                this.fetchSubjects();
            }
        },
        userVerification() {
            const namePattern = /^[a-zA-Z\s]+$/;
            const phonePattern = /^[0-9]+$/;

            const isNameValid = namePattern.test(this.name);
            const isPhoneValid =
                phonePattern.test(this.phone) &&
                this.phone.length >= 7 &&
                this.phone.length <= 15;

            this.isNameValid = isNameValid; // Store name validation result
            this.isPhoneValid = isPhoneValid; // Store phone validation result
            this.isCartNotEmpty = this.cart.length > 0; // Check if the cart is not empty

            // Enable checkout if both name and phone are valid and cart is not empty
            this.isCheckoutEnabled =
                isNameValid && isPhoneValid && this.isCartNotEmpty;
        },

        // alert success message after checkout
        checkout() {
            alert("Order submitted successfully!");
            this.resetForm();
        },

        resetForm() {
            this.name = "";
            this.phone = "";
            this.cart = [];
            this.isCheckoutEnabled = false;
            this.showCheckout = false;
            this.showProduct = true;
            window.location.reload(true);
        },

        performSearch() {
            // Perform search on the server
            fetch(
                `https://afterschoollessons-env.eba-46im9ecw.eu-west-2.elasticbeanstalk.com/search?q=${this.searchKeyword}`
            )
                .then((response) => response.json())
                .then((data) => {
                    console.log("Search results:", data);
                    this.subjects = data; // Update the search results
                })
                .catch((error) => console.error("Error fetching data:", error))
                .finally(() => {
                    // Once the data is loaded, you can set showProduct to true
                    this.showProduct = true;
                });
        },

        // Test Console
        toggleTestConsole: function () {
            this.viewTestConsole = !this.viewTestConsole;
        },
        deleteCaches() {
            caches.keys().then(function (names) {
                for (let name of names) caches.delete(name);
            });
            console.log("All Caches Deleted");
        },
        unregisterServiceWorkers() {
            navigator.serviceWorker
                .getRegistrations()
                .then(function (registrations) {
                    for (let registration of registrations) {
                        registration.unregister();
                    }
                });
            console.log("ServiceWorkers Unregistered");
        },
        reloadPage() {
            window.location.reload(true);
        },
    },

    created() {
        if ("serviceWorker" in navigator) {
            navigator.serviceWorker
                .register("service-worker.js")
                .then((registration) => {
                    console.log(
                        "Service Worker registered with scope:",
                        registration.scope
                    );
                })
                .catch((error) => {
                    console.error("Service Worker registration failed:", error);
                });
        }
    },

    computed: {
        // Counter for cart item
        cartItemCount: function () {
            let totalCount = 0;

            for (const item of this.cart) {
                totalCount += item.Quantity;
            }

            return totalCount || "";
        },
    },
};
</script>

<template>
    <div id="app">
        <!-- Header Section -->
        <header>
            <div class="container1">
                <div class="sub-container1">
                    <h1 v-text="sitename"></h1>
                    <button class="cart" v-on:click="showCart">
                        {{ cartItemCount }}
                        Cart
                    </button>
                </div>
            </div>
        </header>

        <!-- Main Section -->
        <main>
            <!-- Search and Sort section -->
            <div class="search-sort-container" v-if="showProduct">
                <div class="search">
                    <input
                        class="search-input"
                        type="text"
                        id="searchInput"
                        v-model="searchKeyword"
                        @input="performSearch"
                    />
                </div>

                <!-- Sort section -->
                <div class="sort">
                    <div style="max-width: 50em" class="input-group m-auto">
                        <label
                            class="input-group-text"
                            for="inputGroupSelect01"
                        ></label>
                        <select
                            id="inputGroupSelect01"
                            @change="sortSubject"
                            v-model="sortCriteria"
                            class="form-select"
                        >
                            <option selected value="Title_asc">
                                Title (Ascending)
                            </option>
                            <option value="Title_desc">
                                Title (Descending)
                            </option>
                            <option value="Location_asc">
                                Location (Ascending)
                            </option>
                            <option value="Location_desc">
                                Location (Descending)
                            </option>
                            <option value="Price_asc">Price (Ascending)</option>
                            <option value="Price_desc">
                                Price (Descending)
                            </option>
                            <option value="Quantity_asc">
                                Spaces (Ascending)
                            </option>
                            <option value="Quantity_desc">
                                Spaces (Descending)
                            </option>
                        </select>
                    </div>
                </div>
            </div>
            <!-- Test Console Button -->
            <div class="test-console-container" v-if="showProduct">
                <button class="button test-console" @click="toggleTestConsole">
                    Test Console
                </button>
            </div>

            <!-- Test Console Section -->
            <div class="view-TestConsole" v-if="viewTestConsole">
                <!-- Console options -->
                <a :href="url" target="_blank" class="test-button">
                    Accept Exception
                </a>
                <button class="test-button" @click="reloadPage">
                    Reload Page
                </button>
                <button class="test-button" @click="unregisterServiceWorkers">
                    Unregister Service Worker
                </button>
                <button class="test-button" @click="deleteCaches">
                    Delete All Cache
                </button>
            </div>

            <!-- Products Section -->
            <Lesson
                :subjects="subjects"
                :showProduct="showProduct"
                @addToCart="addToCart"
            />

            <!-- Cart page section -->
            <Checkout
                v-if="!showProduct"
                :cart="cart"
                @removeCartItem="removeCartItem"
            ></Checkout>

            <!-- Checkout Section -->
            <div class="checkout-container" v-if="showCheckout">
                <div class="checkout-box">
                    <label for="name">Name:</label>
                    <input
                        type="text"
                        id="name"
                        v-model="name"
                        @input="userVerification"
                    />
                </div>
                <div class="checkout-box">
                    <label for="phone">Phone:</label>
                    <input
                        type="text"
                        id="phone"
                        v-model="phone"
                        @input="userVerification"
                    />
                </div>
                <div class="checkout-box">
                    <button
                        id="checkout"
                        :disabled="!isCheckoutEnabled"
                        @click="checkout"
                    >
                        Checkout
                    </button>
                </div>
            </div>
        </main>
    </div>
</template>
