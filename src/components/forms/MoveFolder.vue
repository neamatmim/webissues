<!--
* This file is part of the WebIssues Server program
* Copyright (C) 2006 Michał Męciński
* Copyright (C) 2007-2017 WebIssues Team
*
* This program is free software: you can redistribute it and/or modify
* it under the terms of the GNU Affero General Public License as published by
* the Free Software Foundation, either version 3 of the License, or
* (at your option) any later version.
*
* This program is distributed in the hope that it will be useful,
* but WITHOUT ANY WARRANTY; without even the implied warranty of
* MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
* GNU Affero General Public License for more details.
*
* You should have received a copy of the GNU Affero General Public License
* along with this program.  If not, see <http://www.gnu.org/licenses/>.
-->

<template>
  <div class="container-fluid">
    <FormHeader v-bind:title="$t( 'MoveFolder.MoveFolder' )" v-on:close="close"/>
    <Prompt path="MoveFolder.MoveFolderPrompt"><strong>{{ name }}</strong></Prompt>
    <FormGroup v-bind:label="$t( 'MoveFolder.Project' )" v-bind:required="true" v-bind:error="locationError">
      <LocationFilters ref="location" v-bind:project="project" v-bind:require-admin="true" v-bind:folder-visible="false" v-on:select-project="selectProject"/>
    </FormGroup>
    <FormButtons v-on:ok="submit" v-on:cancel="cancel"/>
  </div>
</template>

<script>
import { mapState } from 'vuex'

import { ErrorCode } from '@/constants'

export default {
  props: {
    projectId: Number,
    folderId: Number,
    name: String
  },

  data() {
    return {
      selectedProjectId: this.projectId,
      locationError: null
    };
  },

  computed: {
    ...mapState( 'global', [ 'projects' ] ),
    project() {
      if ( this.selectedProjectId != null )
        return this.projects.find( p => p.id == this.selectedProjectId );
      else
        return null;
    }
  },

  methods: {
    selectProject( project ) {
      if ( project != null )
        this.selectedProjectId = project.id;
      else
        this.selectedProjectId = null;
    },

    submit() {
      this.locationError = null;

      const data = { folderId: this.folderId };
      let modified = false;
      let valid = true;

      if ( this.selectedProjectId != null ) {
        if ( this.selectedProjectId != this.projectId )
          modified = true;
        data.projectId = this.selectedProjectId;
      } else {
        this.locationError = this.$t( 'MoveFolder.NoProjectSelected' );
        if ( valid )
          this.$refs.location.focus();
        valid = false;
      }

      if ( !valid )
        return;

      if ( !modified ) {
        this.returnToDetails();
        return;
      }

      this.$emit( 'block' );

      this.$ajax.post( '/server/api/project/folder/move.php', data ).then( ( { changed } ) => {
        if ( changed )
          this.$store.commit( 'global/setDirty' );
        this.returnToDetails();
      } ).catch( error => {
        if ( error.reason == 'APIError' && error.errorCode == ErrorCode.FolderAlreadyExists ) {
          this.$emit( 'unblock' );
          this.locationError = this.$t( 'ErrorCode.' + error.errorCode );
          this.$nextTick( () => {
            this.$refs.location.focus();
          } );
        } else {
          this.$emit( 'error', error );
        }
      } );
    },

    cancel() {
      this.$router.push( 'RenameFolder', { projectId: this.projectId, folderId: this.folderId } );
    },

    returnToDetails() {
      this.$router.push( 'ProjectDetails', { projectId: this.projectId } );
    },

    close() {
      this.$emit( 'close' );
    }
  }
}
</script>