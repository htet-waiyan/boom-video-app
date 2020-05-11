<template>
    <div>
        <div class="video_chat_window" v-if="hasRegistered">
            <div class="row mb-2 mt-2">
              <div class="col text-right">
                &nbsp;
                <!-- <button @click="leaveRoomIfJoined" class="btn btn-sm btn-danger">
                  Leave Meeting
                </button> -->
              </div>
            </div>
            <!-- local track here -->
            <div class="row">
              <div class="col-3">
                <div class="card local-video-window" id="localTrack" style="width: 18rem;">
                  <div class="call-dial-window" v-if="localTrack">
                    <button class="bg-danger text-light btn btn-default rounded-circle"
                      @click="leaveRoomIfJoined">
                      <v-icon name="phone"/></button>
                  </div>
                </div>
              </div>
              <div class="col">
                <div :class="remoteDevCallSize" id="remoteTrack">
                </div>
              </div>
            </div>
        </div>
        <div class="meeting-registration align-middle" v-else>
            <h5 class="text-white mb-2">Join the Daily Standup Call</h5>
            <form class="form-inline">
              <input type="text" class="form-control mb-2 mr-sm-2"
                placeholder="Enter your username"
                v-model="username">
              <button type="submit" class="btn btn-primary mb-2"
                :disabled="!username"
                @click="startDailyStandUp">Join the standup</button>
            </form>
        </div>
    </div>
</template>

<script>
import Video, { createLocalVideoTrack } from 'twilio-video';
import axios from 'axios';
import $ from 'jquery';

export default {
  name: 'BoomVideo',
  data() {
    return {
      username: '',
      roomName: 'Daily Standup',
      identity: '',
      token: '',
      previewTracks: null,
      localMediaAvailable: false,
      hasJoinedRoom: false,
      hasRegistered: false,
      activeRoom: null,
      localTrack: false,
      noOfRemoteParticipants: 0,
    };
  },
  computed: {
    remoteDevCallSize() {
      if (this.noOfRemoteParticipants <= 1) return 'row row-cols-md-1';
      if (this.noOfRemoteParticipants === 2) return 'row row-cols-md-2';
      return 'row row-cols-md-3';
    },
  },
  methods: {
    // attach video/audo tracks to the DOM
    attachTracks(tracks, container) {
      tracks.forEach((track) => {
        const remoteVideo = track.attach();
        const colDiv = $('<div class="col mb-2"></div>');
        const videoCard = $(`
          <div class="card remote-video-window">
          </div>`);
        videoCard.append(remoteVideo);
        colDiv.append(videoCard);
        container.append(colDiv);
        remoteVideo.style.transform = 'scale(-1, 1)';
      });
    },
    attachParticpantTracks(participant, container) {
      const tracks = Array.from(participant.tracks.values());
      this.attachTracks(tracks, container);
    },
    detachTracks(tracks) {
      tracks.forEach((track) => {
        track.detach().forEach((detachedElement) => {
          detachedElement.remove();
        });
      });
    },
    detachParticipantTracks(participant) {
      const tracks = Array.from(participant.tracks.values());
      this.detachTracks(tracks);
    },
    leaveRoomIfJoined() {
      if (this.activeRoom) {
        this.hasRegistered = false;
        this.activeRoom.disconnect();
        const tracks = Array.from(this.activeRoom.localParticipant.tracks.values());
        tracks.forEach((track) => {
          track.unpublish();
        });
      }
    },
    startDailyStandUp() {
      this.hasRegistered = true;
      this.getAccessToken()
        .then((response) => {
          this.token = response.data.token;
          const connectOptions = {
            name: this.roomName,
            audio: true,
          };
          return connectOptions;
        })
        .then((connectOptions) => {
          this.leaveRoomIfJoined();
          Video.connect(this.token, connectOptions)
            .then((room) => {
              console.log('Successfully connected to a room ', this.roomName);
              this.activeRoom = room;
              this.localMediaAvailable = true;
              // document.getElementById('remoteTrack').innerHTML = '';
              room.participants.forEach((participant) => {
                const previewContainer = $('#remoteTrack');
                this.attachParticpantTracks(participant, previewContainer);
              });
              room.on('trackAdded', (track, participant) => {
                console.log(`${participant.identity} added track: ${track.kind}`);
                const previewContainer = $('#remoteTrack');
                this.attachTracks([track], previewContainer);
              });
              room.on('trackRemoved', (track) => {
                this.detachTracks([track]);
              });
              room.on('participantDisconned', (participant) => {
                this.noOfRemoteParticipants -= 1;
                this.detachParticipantTracks(participant);
              });
              room.on('participantConnected', (participant) => {
                this.noOfRemoteParticipants += 1;
                console.log(`Joining: ${participant.identity}`);
                // const previewContainer = document.getElementById('remoteTrack');
                // participant.tracks.forEach((publication) => {
                //   if (publication.isSubscribed) {
                //     const { track } = publication;
                //     previewContainer.appendChild(track.attach());
                //   }
                // });
                // participant.on('trackSubscribed', (track) => {
                //   previewContainer.appendChild(track.attach());
                // });
              });
              if (!this.localTrack) {
                createLocalVideoTrack()
                  .then((track) => {
                    const localVideo = track.attach();
                    const localMediaContainer = document.getElementById('localTrack');
                    localMediaContainer.appendChild(localVideo);
                    localVideo.style.transform = 'scale(-1, 1)';
                    this.localTrack = true;
                  });
              }
            });
        });
    },
    getAccessToken() {
      if (!this.username) {
        alert('Register before joining daily standup'); return undefined;
      }
      return axios.get(`${process.env.VUE_APP_TOKEN_SERVER_URL}/video/token?identity=${this.username}`)
        .then((response) => {
          this.identity = response.data.identity;
          this.token = response.data.token;
          return response;
        });
    },
  },
};
</script>

<style>
  .hidden-version {
    color: #fff;
  }
  .meeting-registration {
    position: absolute !important;
    top: 35%;
    left: 35%;
    width: auto;
    display: inline-block;
  }
  .local-video-window {
    position: relative !important;
    left: 0%;
    bottom: 0%;
    background-color: #485058!important;
  }
  .call-dial-window {
    position: absolute;
    z-index: 1000;
    bottom: 10%;
    left: 40%;
  }
  .remote-video-window {
    background-color: #485058!important;
  }
</style>
