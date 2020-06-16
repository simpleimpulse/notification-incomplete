<template>
    <div id="notificationWidget">
        <div id="notificationIcon" @click="isHidden = !isHidden">
            <div class="notificationBadge" v-if="unreadNotifications.length > 0">
                <span class="unreadNotificationBadge"
                >{{unreadNotifications.length}}</span>
            </div>
            <div v-else class="notificationBadge"></div>
        </div>
        <div v-if="!isHidden" id="notificationDrawer">
            <div v-if="selectedOrg">
                <div>
                    <img
                        :title="selectedOrg.name"
                        :src="require(`../assets/icons/${selectedOrg.avatar}`)"
                    />
                    <span>{{selectedOrg.name}}</span>
                </div>
            </div>
            <div
                v-if="!(unreadNotifications.length > 0)"
                id="noNotifications"
            >No notifications to show</div>
            <ul
                v-if="unreadNotifications.length > 0"
                class='notificationList'
            >
                <li
                    v-for="n in unreadNotifications"
                    :key="n.id"
                    @click="clickHandler(n.id)"
                    :class="[n.isRead ? 'isRead':'']"
                >
                    <div class="notificationItem">
                        <div
                            class="notificationTarget"
                            v-if=" n.target.type == 'campaign'"
                        >{{ n.target.title }}</div>
                        <div class="notificationText">{{ n.text }}</div>
                        <div class="notificationOrgs">
                            <span v-for="o in n.organizations" :key="o.id" class="orgAvatar">
                                <img :title="o.name" :src="require(`../assets/icons/${o.avatar}`)" />
                            </span>
                        </div>
                    </div>
                </li>
            </ul>
            <button class="closeDrawer" @click="isHidden = true">close</button>
        </div>
    </div>
</template>

<script>
    export default {
        name: "NotificationnWidget",
        props: {},
        methods: {
            showToast(notification) {
                // TODO: Show an actual toast for urgent notifications
                // For now, log it to the console
                console.log("Important:", notification.text);
            },
            clickHandler(notificationId) {
                // Get the notification
                let notification = this.notifications.filter(n => {
                    return n.id === notificationId;
                })[0];

                // Emit isRead event for this notification, if it is being read for the first time
                if (!notification.isRead) {
                    this.sendIsRead(notification);
                }

                // Mark the notification as read locally
                notification.isRead = true;

                // TODO: Take the user to the right page
                // For now, console log a possible destination URL
                if (
                    notification.target.type == "campaign" ||
                    notification.target.type == "organization"
                ) {
                    console.log(
                        "Go to page:",
                        `/${notification.target.type}/${notification.target.id}`
                    );
                } else {
                    console.log("Go to user profile");
                }
            },
            // Emit isRead event for the notification
            sendIsRead(notification) {
                this.$root.$emit("isRead", notification);
            }
        },
        data() {
            return {
                isHidden: true, // Used to toggle show/hide the notification drawer
                selectedOrg: null, // Used to filter notifications by organization
                notifications: [] // An array of all notifications obtained from the backend
            };
        },
        computed: {
            // A list of only unread notifications
            unreadNotifications: function() {
                return this.notifications.filter(n => {
                    return !n.isRead;
                });
            },
            // A list of excluded unread notifications that don't belong to selectedOrg
            remainingUnreadNotifications: function() {
                if (!this.selectedOrg) {
                    return [];
                } else {
                    return this.unreadNotifications.filter(n => {
                        let orgList = n.organizations.map(org => {
                            return org.id;
                        });
                        return !orgList.includes(this.selectedOrg.id);
                    });
                }
            }
        },
        mounted() {
            // Listen for a custom notification event from the backend
            this.$root.$on("notify", notification => {
                this.notifications.unshift(notification);
                // If it's an urgent notification, show a toast
                if (notification.type == "urgent") {
                    this.showToast(notification);
                }
            });

            // Listen for orgUpdate events from the OrganizationWidget
            this.$root.$on("orgUpdate", org => {
                this.selectedOrg = org;
            });
        }
    };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    #notificationWidget {
        width: 250px;
        margin: 0 10px;
        box-shadow: 0px 0px 2px;
        border-radius: 5px;
        text-align: left;
        display: inline-block;
        vertical-align: top;
    }
    #notificationWidget:before {
        content: "Notification Widget";
        font-size: 14px;
        text-align: center;
        position: absolute;
        margin-top: -20px;
    }
    #notificationDrawer {
        max-height: 250px;
        background: aliceblue;
        overflow-y: auto;
    }
    #notificationIcon {
        width: calc(100% - 20px);
        display: inline-block;
        padding: 10px;
        text-align: center;
    }
    #notificationIcon:before {
        content: "🔔";
        text-align: center;
        font-size: 32px;
        display: block;
        margin-bottom: -28px;
    }
    .notificationList {
        padding: 10px 0;
    }
    .notificationItem {
        padding: 10px;
        border-bottom: 1px solid gainsboro;
        cursor: pointer;
    }
    .isRead {
        opacity: 0.7;
    }
    .notificationTarget {
        font-weight: bold;
        font-size: 18px;
    }
    #noNotifications {
        padding-top: 10px;
        text-align: center;
    }
    .notificationBadge {
        height: 20px;
        border-radius: 10px;
        font-size: 12px;
        overflow: hidden;
        display: inline-block;
    }
    .unreadNotificationBadge {
        color: white;
        background: tomato;
        display: inline-block;
        width: 20px;
        height: 20px;
        line-height: 20px;
    }
    .notificationOrgs {
        text-align: right;
    }
    .orgAvatar img {
        width: 20px;
        border-radius: 11px;
        vertical-align: middle;
        border: 1px solid;
        margin: 0px 0px 0 -10px;
        background: white;
    }
    .closeDrawer {
        margin: 10px auto;
        display: block;
        text-decoration: underline;
        background: none;
        border: none;
    }
</style>
