<script setup lang="ts">
import { ref, onMounted } from "vue";

import bgImage from "../assets/images/bg.jpg";
import testIcon from "../assets/icons/categories.png";
import github from "../assets/icons/github.png";
//import itch from "../assets/icons/itchio.png";
import rocket from "../assets/icons/rocket.png";
import twitter from "../assets/icons/twitter.png"

//Icons
import whatsapp from "../assets/icons/whatsapp.png";
import email from "../assets/icons/mail.png";
import linkedin from "../assets/icons/linkedin.png";
import telegram from "../assets/icons/telegram.png";

//Apollo client
import { ApolloClient } from "apollo-client";
import { createHttpLink } from "apollo-link-http";
import { InMemoryCache } from "apollo-cache-inmemory";
import { setContext } from "apollo-link-context";
import gql from "graphql-tag";

let counterFetch = ref(0)

const request = async () => {
    try {
        // HTTP connection to the API
        const httpLink = createHttpLink({
            // You should use an absolute URL here
            uri: "https://api.github.com/graphql",
        });

        // Cache implementation
        const cache = new InMemoryCache();

        const authLink = setContext(async (_, { headers }) => {
            // authentication token 
            const token = `${import.meta.env.VITE_TOKEN_GITHUB}`;
            // Return the headers to the context so httpLink can read them
            return {
                headers: {
                    ...headers,
                    authorization: `Bearer ${token}`,
                },
            };
        });

        // Create the apollo client
        const client = new ApolloClient({
            link: authLink.concat(httpLink),
            cache,
        });

        //Get the data 
        const { data } = await client.query({
            query: gql`
            {
                user(login: "rufi512") {
                    pinnedItems(first: 6) {
                        totalCount
                        edges {
                            node {
                                ... on Repository {
                                    name
                                    url
                                    description
                                    homepageUrl
                                    openGraphImageUrl
                                    languages(first: 10) {
                                        edges {
                                            node {
                                                name
                                                color
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        `,
        });
        const formatted = data.user.pinnedItems.edges.map((el: any) => {
            interface LanguagesProgram {
                color: String,
                name: String,
                size: Number

            }
            const repo = el.node
            const languages = el.node.languages.edges.map((lang: any, size: Number) => {
                return { color: lang.node.color, name: lang.node.name, size }
            })
            return { name: repo.name, homepageUrl: repo.homepageUrl, description: repo.description, url: repo.url, image: repo.openGraphImageUrl, languages }
        })
        return formatted
    } catch (e) {
        counterFetch.value += 1;
        console.log(e)
        return []
    }
};

interface Project {
    name: string,
    description: string,
    homepageUrl: string,
    image: string,
    languages: { color: string, name: string, size: Number }[],
    url: string
}

let data = ref<Project[]>([])

let loadPage = ref(false)


onMounted(async () => {
    setTimeout(() => {
        loadPage.value = true
    }, 500);
    const requestData = async () => {
        try {
            const res = await request();
            if(res.length === 0) throw new Error("Data not found");
            data.value = res
        } catch (error) {
            console.log(error)
            data.value = []
            setTimeout(async () => {
                await requestData()
            }, 3000);
        }

    }

    requestData()



});

</script>

<template>
    <main class="home">
        <div class="header">
            <img :src="bgImage" alt="bg-mountain" />
            <div :class="['container-saludated', loadPage ? 'container-saludated-show' : '']">
                <h3>Hi there! i'm</h3>
                <h1>Rafael Amaya</h1>
                <h4>💻 Web developer 🔥</h4>
            </div>
        </div>
        <section class="about">
            <h2>About me</h2>
            <p>
                Web developer dedicated to client-side and server-side programming, my favorite language is JavaScript,
                but I also like to explore new technologies on a daily basis 😋
            </p>
        </section>
        <section class="projects">
            <h2>Projects</h2>
            <div class="indicator-loading" v-if="data.length === 0">
                <h4>Loading Projects...</h4>
                <p v-if="counterFetch >= 2">this is taking a while...</p>
            </div>
            <div class="container-cards" v-if="data.length > 0">
                <div class="card" v-for='(project) in data'>
                    <img class="image-header" :src="project.image" :alt="project.name" />
                    <h3 class="card-title">{{ project.name }}</h3>
                    <p class="card-description">
                        {{ project.description }}
                    </p>

                    <div class="categories-container">
                        <div class="title-icon">
                            <img :src="testIcon" alt="icon" />
                            <h4>Languages</h4>
                        </div>
                        <div class="container-tags">
                            <div class="tag" v-for='(lang) in project.languages'>
                                <span :style="{ background: lang.color }"></span>
                                <span>{{ lang.name }}</span>
                            </div>
                        </div>
                    </div>

                    <div class="container-buttons">
                        <a class="github" :href="project.url" target="_blank">
                            <div>
                                <img :src="github" alt="github-icon" />
                                <p>View code</p>
                            </div>
                        </a>
                        <a :href="project.homepageUrl" class="live" target="_blank" v-if="project.homepageUrl">
                            <div>
                                <img :src="rocket" alt="rocket" />
                                <p>Watch live</p>
                            </div>
                        </a>
                    </div>
                </div>
            </div>
        </section>

        <section class="contact">
            <h2>Contact</h2>
            <div class="container-contacts">
                <div class="social-media">
                    <div class="media">
                        <img :src="whatsapp" alt="whatsapp" />
                        <p>Phone Number</p>
                    </div>
                    <a href="tel:+58 412-081-4697" target="_blank">+58 412-081-4697</a>
                </div>
                <div class="social-media">
                    <div class="media">
                        <img :src="email" alt="email" />
                        <p>Email Address</p>
                    </div>
                    <a href="mailto:rafajos9@gmail.com" target="_blank">Rafajos9@gmail.com</a>
                </div>

                <div class="social-media">
                    <div class="media">
                        <img :src="telegram" alt="telegram" />
                        <p>Telegram</p>
                    </div>
                    <a href="https://t.me/Rufi512" target="_blank">@Rufi512</a>
                </div>

                <div class="social-media">
                    <div class="media">
                        <img :src="twitter" alt="telegram" />
                        <p>Twitter</p>
                    </div>
                    <a href="https://twitter.com/rufi512" target="_blank">@Rufi512</a>
                </div>

                <div class="social-media">
                    <div class="media">
                        <img :src="linkedin" alt="linkedin" />
                        <p>Linkedin</p>
                    </div>
                    <a href="https://www.linkedin.com/in/rafael-amaya-4520051a9/?locale=en_US" target="_blank">Rafael Amaya</a>
                </div>
            </div>
        </section>
    </main>
</template>

<style lang="scss">
@use "../assets/styles/pages/home.scss";
</style>