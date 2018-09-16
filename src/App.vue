<template>
    <div id="app">
        <nav class="panel">
            <div class="panel-heading">
                <div class="columns">
                    <div class="column">
                        <h3>My Channel Subscriptions</h3>
                    </div>
                    <div class="column"></div>
                    <div class="column"></div>
                    <div class="column"></div>
                    <div class="column">
                        <nav class="pagination" role="navigation" aria-label="pagination">
                            <a class="pagination-previous"
                               @click="defineRequest(prevPageToken)"
                               v-show="prevPageToken">Previous</a>
                            <a class="pagination-next"
                               @click="defineRequest(nextPageToken)"
                               v-show="nextPageToken">Next page</a>
                        </nav>
                    </div>
                </div>
            </div>
            <div v-if="hasErrors" class="panel-block has-text-danger">
                <p>Please authorize first.</p>
            </div>
            <article v-else class="panel-block" v-for="(item, index) in items" :key="index">
                <figure class="media-left">
                    <p class="image is-64x64">
                        <img :src="item.snippet.thumbnails.default.url">
                    </p>
                </figure>
                <div class="media-content">
                    <div class="content">
                        <p>
                            <strong>{{ item.snippet.title }}</strong>
                            <br>
                            {{ limitDescText(item.snippet.description) }}
                        </p>
                    </div>
                </div>
            </article>
            <div v-if="hasErrors" class="panel-block">
                <button class="button is-link is-outlined is-fullwidth" @click="handleAuthClick()">
                    Authorize
                </button>
            </div>
        </nav>
    </div>
</template>

<script>
    import clientJSON from '../client_secret.json'

    export default {
        name: 'app',
        data: function () {
            return {
                GoogleAuth: {},
                credentials: clientJSON,
                items: [],
                hasErrors: false,
                nextPageToken: null,
                prevPageToken: null
            }
        },
        methods: {
            injectGapi() {
                let self = this;
                const script = document.createElement("script");
                script.src = "https://apis.google.com/js/client.js";

                script.onload = () => {
                    window.gapi.load('client:auth2', () => {
                        window.gapi.client.init({
                            'clientId': self.credentials.web.client_id,
                            'discoveryDocs': ['https://www.googleapis.com/discovery/v1/apis/youtube/v3/rest'],
                            'scope': 'https://www.googleapis.com/auth/youtube.force-ssl https://www.googleapis.com/auth/youtubepartner'
                        }).then(function () {
                            self.GoogleAuth = window.gapi.auth2.getAuthInstance();

                            self.GoogleAuth.isSignedIn.listen(self.updateSigninStatus());

                            // self.setSigninStatus();
                        });
                    });
                };

                document.body.appendChild(script);
            },

            setSigninStatus() {
                let user = this.GoogleAuth.currentUser.get();
                let isAuthorized = user.hasGrantedScopes('https://www.googleapis.com/auth/youtube.force-ssl https://www.googleapis.com/auth/youtubepartner');
                if (isAuthorized) {
                    this.defineRequest();
                }
            },

            updateSigninStatus() {
                this.setSigninStatus();
            },

            defineRequest(pageToken = '') {
                this.buildApiRequest('GET',
                    '/youtube/v3/subscriptions',
                    {
                        'mine': 'true',
                        'pageToken': this.pageToken(pageToken),
                        'part': 'snippet,contentDetails'
                    });
            },

            buildApiRequest(requestMethod, path, params, properties) {
                params = this.removeEmptyParams(params);
                let request;
                if (properties) {
                    let resource = this.createResource(properties);
                    request = window.gapi.client.request({
                        'body': resource,
                        'method': requestMethod,
                        'path': path,
                        'params': params
                    });
                } else {
                    request = window.gapi.client.request({
                        'method': requestMethod,
                        'path': path,
                        'params': params
                    });
                }
                this.executeRequest(request);
            },

            createResource(properties) {
                let resource = {};
                let normalizedProps = properties;
                for (let p in properties) {
                    let value = properties[p];
                    if (p && p.substr(-2, 2) == '[]') {
                        let adjustedName = p.replace('[]', '');
                        if (value) {
                            normalizedProps[adjustedName] = value.split(',');
                        }
                        delete normalizedProps[p];
                    }
                }
                for (let r in normalizedProps) {
                    // Leave properties that don't have values out of inserted resource.
                    if (normalizedProps.hasOwnProperty(r) && normalizedProps[r]) {
                        let propArray = r.split('.');
                        let ref = resource;
                        for (let pa = 0; pa < propArray.length; pa++) {
                            let key = propArray[pa];
                            if (pa == propArray.length - 1) {
                                ref[key] = normalizedProps[r];
                            } else {
                                ref = ref[key] = ref[key] || {};
                            }
                        }
                    }
                }
                return resource;
            },

            removeEmptyParams(params) {
                for (let p in params) {
                    if (!params[p] || params[p] == 'undefined') {
                        delete params[p];
                    }
                }
                return params;
            },

            executeRequest(request) {
                let self = this;
                request.execute(function (response) {

                    if (response.error) self.hasErrors = true;

                    if (response.nextPageToken) self.nextPageToken = response.nextPageToken;
                    else self.nextPageToken = null;
                    if (response.prevPageToken) self.prevPageToken = response.prevPageToken;
                    else self.prevPageToken = null;

                    self.items = response.items;
                });
            },

            handleAuthClick() {
                this.GoogleAuth.signIn();
            },

            limitDescText(desc) {
                let text = desc.slice(0, 250);

                if (desc.length > 250)
                    text += '...';

                return text;
            },

            pageToken(token) {
                if (token.length === 0) return '';

                return token;
            }

        },
        mounted() {
            this.injectGapi()
        }
    }
</script>