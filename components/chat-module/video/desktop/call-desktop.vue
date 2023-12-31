<template>
    <div class="video-wrapp">
        <div class="video-wrapp-inside">
            <div id="remote-media-div" ref="remote-video" class="remote-video-d"></div>
            <div id="my-video-chat-window" ref="local-video" class="local-video-d"></div>
        </div>
    </div>
</template>

<script>

import {mapGetters} from 'vuex'
import {connect, createLocalVideoTrack} from 'twilio-video'
import chatApi from '@/api/chatApi'

export default {
    props: ['user', 'interlocuitor'],
    data: () => ({
        videoConference: false,
        activeRoom: null,
    }),
    watch: {
        activeRoom() {
            this.activeRoom.localParticipant.videoTracks.forEach(publication => {
                if (!this.camera) {
                    publication.track.disable()
                    document.getElementById('my-video-chat-window').style.display = "none";
                } else {
                    publication.track.enable()
                    document.getElementById('my-video-chat-window').style.display = "block";
                }
            })
        },
        camera() {
            this.activeRoom.localParticipant.videoTracks.forEach(publication => {
                if (!this.camera) {
                    document.getElementById('my-video-chat-window').style.display = "none";
                    publication.track.disable()
                } else {
                    publication.track.enable()
                    document.getElementById('my-video-chat-window').style.display = "block";
                }
            })
        },
        microphone() {
            this.activeRoom.localParticipant.audioTracks.forEach(publication => {
                if (!this.microphone) {
                    publication.track.disable()
                } else {
                    publication.track.enable()
                }
            })
        },
        endChat() {
            this.activeRoom.disconnect()
        }
    },
    computed: mapGetters({
        room: 'call/getRoomId',
        camera: 'chat/getCamera',
        microphone: 'chat/getMicrophone',
        endChat: 'chat/getEndChat',
    }),
    async mounted() {
        $nuxt.$on('endVideoChat', () => {
            this.activeRoom.disconnect()
        })
        await this.startVideConference()
        document.getElementById('my-video-chat-window').style.display = "none";
    },
    methods: {
        cancelVideConference() {
            this.activeRoom.disconnect()
            this.videoConference = false
        },
        startVideConference() {
            this.videoConference = true
            this.getAccessToken()
        },
        async getAccessToken() {
            await chatApi.getTwillioToken(this.user.name, response => {
                this.accessToken = response.token
                this.connectToRoom()
            })
        },
        async connectToRoom() {
            let audioOutputDevice;

            await navigator.mediaDevices.enumerateDevices().then(devices => {
                audioOutputDevice = devices.find(device => device.kind === 'audiooutput');
            })

            await connect(this.accessToken, { name: this.room, type: 'peer-to-peer' }).then(room => {

                console.log(this.room);

                const localVideo = this.$refs["local-video"];
                const remoteVideo = this.$refs["remote-video"];
                this.activeRoom = room;

                console.log(this.activeRoom.participants);
                // if (this.activeRoom.participants.size < 2) {
                    createLocalVideoTrack().then(track => {
                        localVideo.appendChild(track.attach())
                    })

                    room.on('trackAdded', track => {
                        if (track.kind === 'audio') {
                            const audioElement = track.attach();
                            audioElement.setSinkId(audioOutputDevice.deviceId).then(() => {
                                document.body.appendChild(audioElement);
                            })
                        }
                    })

                    room.participants.forEach(participant => {
                        participant.on('trackSubscribed', track => {
                            remoteVideo.appendChild(track.attach())
                        })
                    })

                    room.on('participantConnected', participant => {
                        participant.tracks.forEach(publication => {
                            if (publication.isSubscribed) {
                                const track = publication.track
                                remoteVideo.appendChild(track.attach())
                            }
                        })
                        participant.on('trackSubscribed', track => {
                            remoteVideo.appendChild(track.attach())
                        })
                    })
                // }

                room.on('disconnected', room => {
                    room.localParticipant.videoTracks.forEach(publication => {
                        publication.track.disable()
                        publication.track.stop()
                        publication.unpublish()
                    })
                })
            }, error => {
                console.error(`Unable to connect to Room: ${error.message}`)
            })
        },
    }
}
</script>

<style lang="scss">
.video-wrapp {
    display: flex;
    align-items: center;
    justify-content: center;
}

.video-wrapp-inside {
    position: relative;
}

.remote-video-d {
    max-height: 60vh;

    video {
        width: 100%;
        max-height: 50vh;
    }
}

.local-video-d video {
    position: absolute !important;
    right: 20% !important;
    top: 0 !important;
}

.local-video-d video {
    height: 150px !important;
}

.video-wrapp-inside {
    width: 100%;

    video {
        //width: 100%;
    }
}

.v-item-group.v-bottom-navigation {
    z-index: 9;
}
.my-video-chat-window {
    width: 100% !important;
}
</style>
