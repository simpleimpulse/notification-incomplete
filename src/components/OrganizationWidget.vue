<template>
    <div id="organizationWidget">
        <span id="orgSelector" @click="isHidden = !isHidden">
            <span v-if="selectedOrg">{{ selectedOrg.name }}</span>
            <span v-else>Select an Organization</span>
        </span>
        <div v-if="!isHidden" id="orgDrawer">
            <ul>
                <li v-for="org in organizations" :key="org.id" @click="selectOrg(org.id)">
                    <span class="orgAvatar">
                        <img :src="require(`../assets/icons/${org.avatar}`)" />
                    </span>
                    <span class="orgName">{{ org.name }}</span>
                    <span
                        class="notificationBadge"
                        v-if="org.notificationCount"
                    >{{org.notificationCount}}</span>
                </li>
            </ul>
            <button class="clearSelected" @click="clearSelected">clear</button>
            <button class="closeDrawer" @click="isHidden = true">close</button>
        </div>
    </div>
</template>

<script>
    export default {
        name: "OrganizationWidget",
        props: {},
        methods: {
            // Select an organization
            selectOrg(orgId) {
                this.selectedOrg = this.organizations.filter(org => {
                    return org.id === orgId;
                })[0];
            },

            //Clear selected organization
            clearSelected() {
                this.selectedOrg = null;
            },

            //Emit an orgUpdate event, with the selected Organization
            sendOrgUpdate() {
                this.$root.$emit("orgUpdate", this.selectedOrg);
            }
        },
        data() {
            return {
                isHidden: true, // Toggle the open/close of the widget
                selectedOrg: null, // Used to filter notifications
                organizations: [] // An array of all orgs, obtained from the backend
            };
        },
        created() {
            // Listen for userData from the backend and update organizations accordingly
            this.$root.$on("userData", userData => {
                this.organizations = userData.organizations;
                this.organizations.forEach(function(org) {
                    // Assign it a notification count if one doesn't already exist.
                    org.notificationCount = org.notificationCount || 0;
                });
            });

            // Listen for isRead and update notification count
            this.$root.$on("isRead", notification => {
                // Get a list of just the Org ids for this notification
                const orgList = notification.organizations.map(org => {
                    return org.id;
                });
                // Decrement the notificationCount field on the relevant Organizations
                this.organizations
                    .filter(org => {
                        return orgList.includes(org.id);
                    })
                    .forEach(org => {
                        org.notificationCount -= 1;
                    });

                // TODO: Refactor so forceUpdate() isn't needed.
                // Force the update so badges will update live even when Org picker is open
                this.$forceUpdate();
            });

            //Listen for new notification events
            this.$root.$on("notify", notification => {
                // Get a list of just the Org ids for this notification
                const orgList = notification.organizations.map(org => {
                    return org.id;
                });

                // Increment the notificationCount field on the relevant Organizations
                this.organizations
                    .filter(org => {
                        return orgList.includes(org.id);
                    })
                    .forEach(org => {
                        org.notificationCount += 1;
                    });

                // TODO: Refactor so forceUpdate() isn't needed.
                // Force the update so badges will update live even when Org picker is open
                this.$forceUpdate();
            });
        },
        watch: {
            // Whenever selectedOrg changes, emit an orgUpdate event
            selectedOrg: function() {
                this.sendOrgUpdate();
            }
        }
    };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    #organizationWidget {
        width: 250px;
        margin: 0 10px;
        text-align: left;
        display: inline-block;
        vertical-align: top;
    }
    #organizationWidget:before {
        content: "Organization Widget";
        font-size: 14px;
        text-align: center;
        position: absolute;
        margin-top: -20px;
    }
    #orgSelector {
        width: calc(100% - 22px);
        border: 1px solid #abcabc;
        display: inline-block;
        padding: 17px 10px;
        height: 24px;
        border-radius: 5px;
    }
    #orgSelector:before {
        content: "💼";
        margin: 0 5px;
        text-align: left;
        display: inline-block;
    }
    #orgDrawer {
        background: #abcabc;
    }
    #orgDrawer li {
        margin: 0;
        padding: 10px 13px;
        border-bottom: 1px solid darkseagreen;
    }
    .orgAvatar img {
        width: 24px;
        border-radius: 13px;
        vertical-align: middle;
        border: 1px solid;
        margin: 0 7px 0 0px;
    }
    .notificationBadge {
        background: tomato;
        width: 18px;
        height: 18px;
        border-radius: 5px;
        text-align: center;
        color: white;
        font-size: 11px;
        line-height: 18px;
        float: right;
    }
    .closeDrawer,
    .clearSelected {
        margin: auto;
        display: block;
        text-decoration: underline;
        background: none;
        border: none;
    }
    .clearSelected {
        margin: 10px auto;
    }
</style>
