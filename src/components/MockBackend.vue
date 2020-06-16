<template>
    <div id="mock-backend">
        <p>
            Pick a campaign to trigger a notification on it.
            <br />
            <small>Organization level notifications will pick a random org for you.</small>
        </p>
        <ul id="campaignSelector">
            <li
                v-for="campaign in campaigns"
                v-bind:key="campaign.id"
                @click="selectCampaign(campaign.id)"
                :class="{ 'active': selectedCampaign != null && selectedCampaign.id === campaign.id }"
            >{{ campaign.title }}</li>
        </ul>
        <div>
            <p>Click to send a notification:</p>
            <button
                v-for="notification in notificationTemplates"
                v-bind:key="notification.id"
                @click="selectNotificationTemplate(notification.id)"
                :disabled="notification.target == 'campaign' && selectedCampaign == null"
            >{{ notification.cta }}</button>
        </div>
        <div class="text-left">
            <p>
                <strong>Info:</strong>
                <br />Under the hood, Campaigns are linked to Organizations as follows:
                <br />Campaign A : Org A, Org B, Org C
                <br />Campaign B : Org B, Org D
                <br />Campaign C : Org B, Org C
                <br />Campaign D : Org B, Org D
                <br />
                <br />The User is associated with Org A, Org B &amp; Org C
            </p>
        </div>
    </div>
</template>

<script>
    /* This section will act as a standin for the DB. */
    // All data on here is blackboxed and can only be communicated to the $root via events.

    // Notification data for all possible notifications
    const notificationTemplates = [
        {
            id: "campaign-paused",
            cta: "Campaign: Paused",
            text: "Your Campaign has been paused",
            target: "campaign",
            type: "urgent"
        },
        {
            id: "campaign-approved",
            cta: "Campaign: Approved",
            text: "You have been approved on this Campaign.",
            target: "campaign",
            type: "urgent"
        },
        {
            id: "new-invoice",
            cta: "Organization: New Invoice",
            text: "A new invoice is available",
            target: "organization",
            type: "urgent"
        },
        {
            id: "new-feature",
            cta: "Organization: New Feature",
            text: "A new Feature X has been released.",
            target: "organization",
            type: "optional"
        },
        {
            id: "referral-success",
            cta: "User: Referral Success",
            text: "Your referral link was used for a new sign up",
            target: "user",
            type: "info"
        }
    ];

    // Organization data for all existing orgs
    const organizationData = [
        {
            id: 123,
            name: "Organization A",
            avatar: "orgA.png"
        },
        {
            id: 234,
            name: "Organization B",
            avatar: "orgB.png"
        },
        {
            id: 345,
            name: "Organization C",
            avatar: "orgC.png"
        },
        {
            id: 456,
            name: "Organization D",
            avatar: "orgD.png"
        }
    ];

    // Campaign data for all existing campaigns
    const campaignData = [
        {
            id: 1,
            title: "Campaign A",
            orgs: [123, 234, 456]
        },
        {
            id: 2,
            title: "Campaign B",
            orgs: [234, 456]
        },
        {
            id: 3,
            title: "Campaign C",
            orgs: [234, 345]
        },
        {
            id: 4,
            title: "Campaign D",
            orgs: [234, 456]
        }
    ];

    // Mock user data for the current frontend user
    const user = {
        id: 1,
        name: "Michael Scott",
        organizations: [123, 234, 345]
    };

    /* Business logic & Utilities */

    // Given an action target,
    // return the target object id & type.
    const getTargetObject = action => {
        let target = {};
        // If it's an organization level action, return the target organization object
        if (action.target == "organization") {
            target = organizationData.filter(org => {
                return org.id === action.id;
            })[0];
        }

        // If it's a campaign level action return the target campaign object
        else if (action.target == "campaign") {
            target = campaignData.filter(campaign => {
                return campaign.id === action.id;
            })[0];
        }

        target.type = action.target;
        return target;
    };

    // Given an action target and user,
    // return an array of user's organizations
    // that the action would apply to.
    const getOrganizations = (action, user) => {
        let orgList = []; // This will be filled with relevant org ids, then used to filter organizationData

        // If it's an organization level action, just return the target organization
        if (action.target == "organization") {
            orgList = organizationData
                .filter(org => {
                    return org.id === action.id;
                })
                .map(org => {
                    return org.id;
                });
        }

        // If it's a campaign level action return all of the user's orgs affected by it
        else if (action.target == "campaign") {
            // Get all orgs the campaign is associated with
            const campaignOrgs = campaignData.filter(campaign => {
                return campaign.id === action.id;
            })[0].orgs;

            orgList = user.organizations.filter(org => campaignOrgs.includes(org));
        }
        return organizationData.filter(org => {
            return orgList.includes(org.id);
        });

        // Note: If it's a user level action, no organizations are involved and orgList can stay empty
    };

    // A class to construct new Notification objects
    class Notification {
        constructor(id, actionType, target) {
            var notificationTemplate = notificationTemplates.filter(n => {
                return n.id === actionType;
            })[0];
            this.id = id;
            this.type = notificationTemplate.type;
            this.text = notificationTemplate.text;
            this.target = getTargetObject(target);
            this.organizations = getOrganizations(target, user);
            this.isRead = false;
        }
    }

    /* The Vue model that provides a handy control panel to trigger notifications to the frontend */
    export default {
        name: "MockBackend",
        methods: {
            //Update selectedCampaign when a campaign is selected
            selectCampaign(campaignId) {
                this.selectedCampaign = this.campaigns.filter(campaign => {
                    return campaign.id === campaignId;
                })[0];
            },
            // Update selectedNotificationTemplate when a notification template is selected
            selectNotificationTemplate(notificationId) {
                this.selectedNotificationTemplate = notificationTemplates.filter(
                    notification => {
                        return notification.id === notificationId;
                    }
                )[0];
                this.triggerNotification();
            },
            // Based on the selected campaign and notification template,
            // create a valid Notification object and emit a notify event with it.
            triggerNotification() {
                let action = {
                    target: this.selectedNotificationTemplate.target
                };

                if (action.target === "campaign") {
                    action.id = this.selectedCampaign.id;
                } else if (action.target === "organization") {
                    //Assign a random organization from the user's list of orgs
                    action.id =
                        user.organizations[
                            (user.organizations.length * Math.random()) | 0
                        ];
                } else {
                    //Assign the current user's id
                    action.id = user.id;
                }

                const notification = new Notification(
                    this.lastNotificationId,
                    this.selectedNotificationTemplate.id,
                    action
                );
                // Increment lastNotificationId (Would be automatic when using a real db)
                this.lastNotificationId += 1;

                // Emit the notify event
                this.$root.$emit("notify", notification);
            },
            // Substitute for an API to get User / Org data
            // Gather user information from the DB and emit the userData event
            sendUserData() {
                const userData = {
                    id: user.id,
                    name: user.name,
                    organizations: organizationData.filter(org => {
                        return user.organizations.includes(org.id);
                    })
                };
                this.$root.$emit("userData", userData);
            }
        },
        data() {
            return {
                selectedCampaign: null,
                selectedNotificationTemplate: null,
                campaigns: campaignData,
                notificationTemplates: notificationTemplates,
                lastNotificationId: 0 // This will be incremented each time a new notification is created
            };
        },
        mounted() {
            this.sendUserData();
        }
    };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    #campaignSelector {
        padding: 0;
        margin: 0;
        font-size: 0.8em;
        list-style: none;
    }
    #campaignSelector li {
        border: 1px solid #abcabc;
        padding: 10px;
        display: inline-block;
        cursor: pointer;
    }
    .active {
        background: #abcabc;
    }
    button {
        margin: 10px auto;
        padding: 10px;
        display: block;
        width: 100%;
    }
    .text-left {
        text-align: left;
    }
</style>
