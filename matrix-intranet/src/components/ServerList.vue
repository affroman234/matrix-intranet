<template>
    <div>
        <h1 class="title is-1">
                 Matrix Development Server Control Panel
        </h1>
        <table class="table table--centered is-striped has-margin-top has-margin-bottom">
            <thead>
                <th>Server Name</th>
                <th>Server PowerSaver</th>
                <th>Server State</th>
            </thead>
                <tr v-for="(instance, index) in reservations.Reservations">
                    <td>{{devServers.instanceNames[index]}}</td>
                    <td>{{devServers.powerSaver[index]}}</td>
                    <td>{{devServers.instanceStatuses.state[index]}}</td>
                    <td>
                        <label class="switch">
                            <input type="checkbox" :checked="devServers.instanceStatuses.checked[index]" @change="displayWarning(index)">
                            <span class="slider round"></span>
                        </label>
                    </td>
                </tr>
        </table>
        <div v-if="showingModal" id="myModal" class="modal">
            <div class="modal-content">
                <span class=" button close is-danger-link" @click="showingModal=false">&times;</span>
                <div class="modal-header">
                    Warning!
                </div>
                <p v-if="devServers.instanceStatuses.checked[selectedIndex]">
                    Are you sure that you want to turn <b> off {{devServers.instanceNames[selectedIndex]}}</b>. This will take a couple of seconds. Please be patient.
                </p>
                <p v-else>
                    Are you sure that you want to turn <b> on  {{devServers.instanceNames[selectedIndex]}}</b>. This will take a couple of seconds. Please be patient.
                </p>
                <button class="button is-danger-link m-r-3" @click="toggleServerState(devServers.instanceStatuses.state[selectedIndex], selectedIndex); showingModal=false">
                    Yes
                </button>
                <button class="button is-danger-link" @click="showingModal=false">
                    No
                </button>
            </div>
        </div>
    </div>
</template>

<script>
import Login from './Login'
import AWS from 'aws-sdk'

export default {
    name: 'ServerList',
    data () {
        return {
            isLoaded: false,
            ec2: null,
            AWScreds: null,
            reservations: {Reservations:null},
            devServers: {
                instanceStatuses: {
                    state: [],
                    checked: []
                },
                instanceIds: [],
                instanceNames: [],
                powerSaver: []
            },
            params: null,
            showingModal: false,
            selectedIndex: null
        }
    },
    mounted () {
        let _this = this
        var elem = document.getElementById("app")
        elem.addEventListener('signIn', function (e) {
            AWS.config.credentials = new AWS.CognitoIdentityCredentials({
                IdentityPoolId: 'us-east-1:22eff6b6-8d19-489a-a954-8a1b6758a80b',
                Logins: {
                    'accounts.google.com': e.detail
                },
            })

            AWS.config.update({region: 'us-east-1'})
            _this.ec2 = new AWS.EC2()
            _this.params = {
                Filters : [{
                    Name: 'tag:environment',
                    Values: [
                        'development'
                    ]
                }]
            }
            _this.update()
            setInterval(_this.update,2000)
            _this.AWScreds = AWS.config.credentials
        }, false)
    },
    methods: {
        getInstanceIds () {
            let _this = this
            this.reservations.Reservations.forEach( function(reserv, index){
                _this.devServers.instanceIds[index] = reserv.Instances[0].InstanceId
            })
        },
        getServerName () {     
            let _this = this
            let tagArray = []
            this.reservations.Reservations.forEach( function(reserv, index){
                tagArray = reserv.Instances[0].Tags
                for (var tag of tagArray) {
                    if (tag.Key === 'Name') {
                        _this.devServers.instanceNames[index] = tag.Value
                    }
                }
            })
        },
        getServerPowerSaver () {
            let _this = this
            let tagArray = []
            this.reservations.Reservations.forEach( function(reserv, index){
                tagArray = reserv.Instances[0].Tags
                for (var tag of tagArray) {
                    if (tag.Key === 'PowerSaver') {
                        if(tag.Value !== 'yes'){
                            _this.devServers.powerSaver[index] = 'No'
                        }
                        else {
                            _this.devServers.powerSaver[index] = 'Yes'
                        }
                    }
                }
            })
        },
        getServerStatus () {
            let _this = this
            this.reservations.Reservations.forEach( function(reserv, index){
                let status = reserv.Instances[0].State.Name
                _this.devServers.instanceStatuses.state[index] = status
                if(status === 'running') _this.devServers.instanceStatuses.checked[index] = true
                else {
                    _this.devServers.instanceStatuses.checked[index] = false
                }
            })
        },
        toggleServerState (state, index) {
            var params = {
                InstanceIds: [this.devServers.instanceIds[index]],
                DryRun: true
            }
            var ec2 = this.ec2
            if(state === 'stopped') {
                this.ec2.startInstances(params, function(err, data) {
                    if (err && err.code === 'DryRunOperation') {
                        params.DryRun = false;
                        ec2.startInstances(params, function(err, data) {
                            if (err) {
                                console.log("Error", err);
                            } else if (data) {
                                console.log("Success", data.StartingInstances);
                            }
                        });
                    } else {
                    console.log("You don't have permission to start instances.");
                    }
                });
            } else if (state === 'running') {
                // call EC2 to stop the selected instances
                this.ec2.stopInstances(params, function(err, data) {
                    if (err && err.code === 'DryRunOperation') {
                    params.DryRun = false;
                    ec2.stopInstances(params, function(err, data) {
                        if (err) {
                            console.log("Error", err);
                        } else if (data) {
                            console.log("Success", data.StoppingInstances);
                        }
                    });
                    } else {
                    console.log("You don't have permission to stop instances.");
                    }
                })
            }
        },
        displayWarning (index) {
            this.showingModal = true
            this.selectedIndex = index
        },
        sortData (data) {
            data.Reservations.sort(function(a, b) {
                let _this = this
                let tagArrayA = a.Instances[0].Tags
                let tagArrayB = b.Instances[0].Tags
                let aName = ''
                let bName = ''
                for (var tag of tagArrayA) {
                    if (tag.Key === 'Name') {
                        aName = tag.Value
                    }
                }
                for (var tag of tagArrayB) {
                    if (tag.Key === 'Name') {
                        bName = tag.Value
                    }
                }
                if (aName < bName) return -1
                if (aName > bName) return 1
                return 0
            })
            return data
        },
        update () {
            let _this = this
            this.ec2.describeInstances(_this.params, function (err, data) {
                if (err) console.log(err, err.stack)
                else {
                    _this.reservations = _this.sortData(data)
                    _this.getInstanceIds()
                    _this.getServerName()
                    _this.getServerStatus()
                    _this.getServerPowerSaver()
                }
            })
        }
    }
}
</script>