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
    <FormHeader v-bind:title="name" v-on:close="close">
      <button type="button" class="btn btn-default" v-bind:title="$t( 'ProjectDetails.Return' )" v-on:click="returnToList"><span class="fa fa-arrow-left" aria-hidden="true"></span></button>
      <DropdownButton v-if="isAdministrator" fa-class="fa-ellipsis-v" menu-class="dropdown-menu-right" v-bind:title="$t( 'ProjectDetails.More' )">
        <li><HyperLink><span class="fa fa-pencil" aria-hidden="true"></span> {{ $t( 'ProjectDetails.RenameProject' ) }}</HyperLink></li>
        <li><HyperLink><span class="fa fa-clock-o" aria-hidden="true"></span> {{ $t( 'ProjectDetails.ArchiveProject' ) }}</HyperLink></li>
        <li><HyperLink><span class="fa fa-trash" aria-hidden="true"></span> {{ $t( 'ProjectDetails.DeleteProject' ) }}</HyperLink></li>
        <li role="separator" class="divider"></li>
        <li><HyperLink v-on:click="projectPermissions"><span class="fa fa-unlock-alt" aria-hidden="true"></span> {{ $t( 'ProjectDetails.Permissions' ) }}</HyperLink></li>
      </DropdownButton>
    </FormHeader>
    <FormSection v-bind:title="$t( 'ProjectDetails.Description' )">
      <DropdownButton v-if="isAdministrator && description != null" fa-class="fa-ellipsis-v" menu-class="dropdown-menu-right" v-bind:title="$t( 'ProjectDetails.Menu' )">
        <li><HyperLink><span class="fa fa-pencil" aria-hidden="true"></span> {{ $t( 'ProjectDetails.Edit' ) }}</HyperLink></li>
        <li><HyperLink><span class="fa fa-trash" aria-hidden="true"></span> {{ $t( 'ProjectDetails.Delete' ) }}</HyperLink></li>
      </DropdownButton>
      <button v-else-if="isAdministrator" type="button" class="btn btn-default">
        <span class="fa fa-pencil" aria-hidden="true"></span> {{ $t( 'ProjectDetails.Add' ) }}
      </button>
    </FormSection>
    <div v-if="description != null" class="description-panel">
      <div class="formatted-text" v-hljs="description.text"></div>
      <div v-if="description.modifiedDate" class="last-edited">
        <span class="fa fa-pencil" aria-hidden="true"></span> {{ description.modifiedDate }} &mdash; {{ description.modifiedBy }}
      </div>
    </div>
    <div v-else class="alert alert-info">
      {{ $t( 'ProjectDetails.NoDescription' ) }}
    </div>
    <FormSection v-bind:title="$t( 'ProjectDetails.Folders' )">
      <button v-if="isAdministrator" type="button" class="btn btn-success">
        <span class="fa fa-plus" aria-hidden="true"></span> {{ $t( 'ProjectDetails.Add' ) }}
      </button>
    </FormSection>
    <Grid v-bind:items="folders" v-bind:column-names="columnNames" v-bind:column-classes="[ 'column-wide', null ]">
      <template slot-scope="{ item, columnIndex, columnClass }">
        <td v-bind:class="columnClass">{{ getCellValue( columnIndex, item ) }}</td>
      </template>
    </Grid>
  </div>
</template>

<script>
import { mapState, mapGetters } from 'vuex'

import { Access } from '@/constants'

export default {
  props: {
    projectId: Number,
    name: String,
    access: Number,
    description: Object,
    folders: Array
  },
  computed: {
    ...mapState( 'global', [ 'types' ] ),
    isAdministrator() {
      return this.access == Access.AdministratorAccess;
    },
    columnNames() {
      return [
        this.$t( 'ProjectDetails.Name' ),
        this.$t( 'ProjectDetails.Type' )
      ];
    }
  },
  methods: {
    getCellValue( columnIndex, folder ) {
      switch ( columnIndex ) {
        case 0:
          return folder.name;
        case 1:
          const type = this.types.find( t => t.id == folder.typeId );
          if ( type != null )
            return type.name;
          break;
      }
    },
    projectPermissions() {
      this.$router.push( 'ProjectPermissions', { projectId: this.projectId } );
    },
    returnToList() {
      this.$router.push( 'ManageProjects' );
    },
    close() {
      this.$emit( 'close' );
    }
  }
}
</script>