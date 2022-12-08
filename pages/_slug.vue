<template>
    <div>
    <div id="menu">
        <ul>
            <li>
                <a href="#">Home</a>
            </li>
            <li>
                <a href="#">News</a>
            </li>
            <li>
                <a href="#">Contract</a>
            </li>
            
        </ul>
    </div>
    <div id="container">
        <h2 id="title">{{this.metadata.title}}</h2>
        <div id="content" v-html="metadata.content"></div>
    </div>
</div>
</template>

<script>
const util = require('util')

const getPostId = (param) => param.substring(param.lastIndexOf('-') + 1);

export default {
    validate({ params }) {
        console.log('validate');
        return params.slug;
    },


    async asyncData({ error, params, $axios, req, redirect }) {
        const postId = getPostId(params.slug)
        console.log(`👉 Request header: 
        ${util.inspect(req.headers, { showHidden: false, depth: null, colors: true })} 
        👉 Request url: ${req.url}`);

        try {
            const metadata = await $axios.$get(`/api.php?post_id=${postId}`);
            // catch facebook agent, stay on this page
            // let userAgent = req?.headers?.user-agent || null;
            // if (userAgent && (userAgent.includes('facebookexternalhit') || userAgent.includes('facebookcatalog'))) {
            // } else {
            // catch from referer, user come from facebook => redirect 
            let referer = req.headers?.referer || null;
            if (referer && referer.includes('facebook.com')) {
                redirect(metadata.link)
            }
            // }
            return { metadata, path: req.url }
        } catch (e) {
            console.log("Error: ", e);
            error({ statusCode: 404, message: 'Post not found' })
        }
    },

    head() {
        return {
            title: this.metadata.title,
            meta: [
                {
                    hid: 'og-title',
                    property: 'og:title',
                    content: this.metadata.title,
                },
                {
                    hid: 'og-type',
                    property: 'og:type',
                    content: 'article',
                },
                {
                    hid: 'og-url',
                    property: 'og:url',
                    content: this.path,
                },
                {
                    hid: 'og-description',
                    property: 'og:description',
                    content: '...',
                },

                {
                    hid: 'og-image',
                    property: 'og:image',
                    content: this.metadata.featured_img_url,
                },

            ]
        }
    }
}
</script>

<style>
#container {
    display: flex;
    flex-direction: column;
    align-items: center;
}

#content {
    width: 70%;
}

img {
    width: 100%;
    height: auto;
}

#show-more {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: -64px;
    height: 64px;
    background-image: linear-gradient(to bottom, rgb(255 0 0 / 0%), rgb(255 255 255), rgb(255 255 255));
}

.btn {
    font-weight: bold;
    font-size: 1em;
    letter-spacing: 0.1em;
    text-decoration: none;
    color: #000000;
    display: inline-block;
    padding: 10px 40px 10px 40px;
    position: relative;
    border: 3px solid #ececec;
    border-radius: 20px;
}

#menu ul {
  list-style-type: none;
  margin: 0;
  padding: 0;
  overflow: hidden;
}

#menu li {
  float: left;
}

#menu li a {
  display: block;
  text-align: center;
  padding: 8px;
  text-decoration: none;
  font-weight: 600;
  color: #000000;
}

@media screen and (max-width: 640px) {
    #content {
        width: 100%;
    }
}
</style>