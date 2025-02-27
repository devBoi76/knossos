<template>
  <div>
    <ModalCreation ref="modal_creation" />
    <ModalReport ref="modal_report" :item-id="user.id" item-type="user" />
    <div class="user-header-wrapper">
      <div class="user-header">
        <Avatar
          :src="previewImage ? previewImage : user.avatar_url"
          size="md"
          circle
          :alt="user.username"
        />
        <h1 class="username">{{ user.username }}</h1>
      </div>
    </div>
    <div class="normal-page">
      <div class="normal-page__sidebar">
        <div class="card sidebar">
          <h1 class="mobile-username">{{ user.username }}</h1>
          <div class="card__overlay">
            <FileInput
              v-if="isEditing"
              :max-size="262144"
              :show-icon="true"
              accept="image/png,image/jpeg,image/gif,image/webp"
              class="choose-image iconified-button"
              prompt="Upload avatar"
              @change="showPreviewImage"
            >
              <UploadIcon />
            </FileInput>
            <button
              v-else-if="$auth.user && $auth.user.id === user.id"
              class="iconified-button"
              @click="isEditing = true"
            >
              <EditIcon />
              Edit
            </button>
            <button
              v-else-if="$auth.user"
              class="iconified-button"
              @click="$refs.modal_report.show()"
            >
              <ReportIcon aria-hidden="true" />
              Report
            </button>
            <a
              v-else
              class="iconified-button"
              :href="authUrl"
              rel="noopener noreferrer nofollow"
            >
              <ReportIcon aria-hidden="true" />
              Report
            </a>
          </div>
          <template v-if="isEditing">
            <div class="inputs universal-labels">
              <label for="user-username"
                ><span class="label__title">Username</span></label
              >
              <input
                id="user-username"
                v-model="user.username"
                maxlength="39"
                type="text"
              />
              <label for="user-bio"
                ><span class="label__title">Bio</span></label
              >
              <div class="textarea-wrapper">
                <textarea
                  id="user-bio"
                  v-model="user.bio"
                  maxlength="160"
                ></textarea>
              </div>
            </div>
            <div class="button-group">
              <button
                class="iconified-button"
                @click="
                  isEditing = false
                  user = JSON.parse(JSON.stringify($auth.user))
                  previewImage = null
                  icon = null
                "
              >
                <CrossIcon /> Cancel
              </button>
              <button
                class="iconified-button brand-button"
                @click="saveChanges"
              >
                <SaveIcon /> Save
              </button>
            </div>
          </template>
          <template v-else>
            <div class="sidebar__item">
              <Badge
                v-if="user.role === 'admin' || user.role === 'moderator'"
                :type="user.role"
              />
              <Badge v-else-if="projects.length > 0" type="creator" />
            </div>
            <span v-if="user.bio" class="sidebar__item bio">{{
              user.bio
            }}</span>
            <hr class="card-divider" />
            <div class="primary-stat">
              <DownloadIcon class="primary-stat__icon" aria-hidden="true" />
              <div class="primary-stat__text">
                <span class="primary-stat__counter">{{ sumDownloads() }}</span>
                downloads
              </div>
            </div>
            <div class="primary-stat">
              <HeartIcon class="primary-stat__icon" aria-hidden="true" />
              <div class="primary-stat__text">
                <span class="primary-stat__counter">{{ sumFollows() }}</span>
                followers of projects
              </div>
            </div>
            <div class="stats-block__item secondary-stat">
              <SunriseIcon class="secondary-stat__icon" aria-hidden="true" />
              <span
                v-tooltip="
                  $dayjs(user.created).format('MMMM D, YYYY [at] h:mm:ss A')
                "
                class="secondary-stat__text date"
              >
                Joined {{ $dayjs(user.created).fromNow() }}
              </span>
            </div>
            <hr class="card-divider" />
            <div class="stats-block__item secondary-stat">
              <UserIcon class="secondary-stat__icon" aria-hidden="true" />
              <span class="secondary-stat__text">
                User ID: <CopyCode :text="user.id" />
              </span>
            </div>
            <a
              v-if="githubUrl"
              :href="githubUrl"
              :target="$external()"
              rel="noopener noreferrer nofollow"
              class="sidebar__item github-button iconified-button"
            >
              <GitHubIcon aria-hidden="true" />
              View GitHub profile
            </a>
          </template>
        </div>
      </div>
      <div class="normal-page__content">
        <Advertisement type="banner" small-screen="square" />
        <nav class="navigation-card">
          <NavRow
            query="type"
            :links="[
              {
                label: 'all',
                href: '',
              },
              ...projectTypes.map((x) => {
                return {
                  label: $formatProjectType(x) + 's',
                  href: x,
                }
              }),
            ]"
          />
          <div class="input-group">
            <NuxtLink
              v-if="$auth.user && $auth.user.id === user.id"
              class="iconified-button"
              to="/dashboard/projects"
            >
              <SettingsIcon />
              Manage projects
            </NuxtLink>
            <button
              v-tooltip="
                $capitalizeString($cosmetics.searchDisplayMode.user) + ' view'
              "
              :aria-label="
                $capitalizeString($cosmetics.searchDisplayMode.user) + ' view'
              "
              class="square-button"
              @click="cycleSearchDisplayMode()"
            >
              <GridIcon v-if="$cosmetics.searchDisplayMode.user === 'grid'" />
              <ImageIcon
                v-else-if="$cosmetics.searchDisplayMode.user === 'gallery'"
              />
              <ListIcon v-else />
            </button>
          </div>
        </nav>
        <div
          v-if="projects.length > 0"
          class="project-list"
          :class="'display-mode--' + $cosmetics.searchDisplayMode.user"
        >
          <ProjectCard
            v-for="project in ($route.query.type !== undefined
              ? projects.filter((x) => x.project_type === $route.query.type)
              : projects
            )
              .slice()
              .sort((a, b) => b.downloads - a.downloads)"
            :id="project.slug || project.id"
            :key="project.id"
            :name="project.title"
            :display="$cosmetics.searchDisplayMode.user"
            :featured-image="
              project.gallery
                .slice()
                .sort((a, b) => b.featured - a.featured)
                .map((x) => x.url)[0]
            "
            :description="project.description"
            :created-at="project.published"
            :updated-at="project.updated"
            :downloads="project.downloads.toString()"
            :follows="project.followers.toString()"
            :icon-url="project.icon_url"
            :categories="project.categories"
            :client-side="project.client_side"
            :server-side="project.server_side"
            :status="
              $auth.user &&
              ($auth.user.id === user.id ||
                $auth.user.role === 'admin' ||
                $auth.user.role === 'moderator')
                ? project.status
                : null
            "
            :has-mod-message="project.moderator_message"
            :type="project.project_type"
            :color="project.color"
          />
        </div>
        <div v-else class="error">
          <UpToDate class="icon" /><br />
          <span v-if="$auth.user && $auth.user.id === user.id" class="text">
            You don't have any projects.<br />
            Would you like to
            <a class="link" @click.prevent="$refs.modal_creation.show()">
              create one</a
            >?
          </span>
          <span v-else class="text">This user has no projects!</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import ProjectCard from '~/components/ui/ProjectCard'
import Badge from '~/components/ui/Badge'
import Advertisement from '~/components/ads/Advertisement'

import GitHubIcon from '~/assets/images/utils/github.svg?inline'
import ReportIcon from '~/assets/images/utils/report.svg?inline'
import SunriseIcon from '~/assets/images/utils/sunrise.svg?inline'
import DownloadIcon from '~/assets/images/utils/download.svg?inline'
import SettingsIcon from '~/assets/images/utils/settings.svg?inline'
import UpToDate from '~/assets/images/illustrations/up_to_date.svg?inline'
import UserIcon from '~/assets/images/utils/user.svg?inline'
import EditIcon from '~/assets/images/utils/edit.svg?inline'
import HeartIcon from '~/assets/images/utils/heart.svg?inline'
import CrossIcon from '~/assets/images/utils/x.svg?inline'
import SaveIcon from '~/assets/images/utils/save.svg?inline'
import GridIcon from '~/assets/images/utils/grid.svg?inline'
import ListIcon from '~/assets/images/utils/list.svg?inline'
import ImageIcon from '~/assets/images/utils/image.svg?inline'
import UploadIcon from '~/assets/images/utils/upload.svg?inline'
import FileInput from '~/components/ui/FileInput'
import ModalReport from '~/components/ui/ModalReport'
import ModalCreation from '~/components/ui/ModalCreation'
import NavRow from '~/components/ui/NavRow'
import CopyCode from '~/components/ui/CopyCode'
import Avatar from '~/components/ui/Avatar'

export default {
  auth: false,
  components: {
    Avatar,
    CopyCode,
    NavRow,
    ModalCreation,
    ModalReport,
    FileInput,
    ProjectCard,
    SunriseIcon,
    DownloadIcon,
    GitHubIcon,
    ReportIcon,
    Badge,
    SettingsIcon,
    UpToDate,
    UserIcon,
    EditIcon,
    Advertisement,
    HeartIcon,
    CrossIcon,
    SaveIcon,
    GridIcon,
    ListIcon,
    ImageIcon,
    UploadIcon,
  },
  async asyncData(data) {
    try {
      const [user, projects] = (
        await Promise.all([
          data.$axios.get(`user/${data.params.id}`, data.$defaultHeaders()),
          data.$axios.get(
            `user/${data.params.id}/projects`,
            data.$defaultHeaders()
          ),
        ])
      ).map((it) => it.data)

      if (user.username !== data.params.id) {
        data.redirect(301, `/user/${user.username}`)

        return
      }

      let gitHubUser = {}
      let versions = []
      try {
        const [gitHubUserData, versionsData] = (
          await Promise.all([
            data.$axios.get(`https://api.github.com/user/` + user.github_id),
            data.$axios.get(
              `versions?ids=${JSON.stringify(
                [].concat.apply(
                  [],
                  projects.map((x) => x.versions)
                )
              )}`
            ),
          ])
        ).map((it) => it.data)
        gitHubUser = gitHubUserData
        versions = versionsData
      } catch {}

      for (const version of versions) {
        const projectIndex = projects.findIndex(
          (x) => x.id === version.project_id
        )
        if (projects[projectIndex].loaders) {
          for (const loader of version.loaders) {
            if (!projects[projectIndex].loaders.includes(loader)) {
              projects[projectIndex].loaders.push(loader)
            }
          }
        } else {
          projects[projectIndex].loaders = version.loaders
        }
      }
      for (const project of projects) {
        project.categories = project.categories.concat(project.loaders)
        project.project_type = data.$getProjectTypeForUrl(
          project.project_type,
          project.categories
        )
      }

      return {
        user,
        projects,
        githubUrl: gitHubUser.html_url,
      }
    } catch {
      data.error({
        statusCode: 404,
        message: 'User not found',
      })
    }
  },
  data() {
    return {
      isEditing: false,
      icon: null,
      previewImage: null,
    }
  },
  head() {
    const description = this.user.bio
      ? `${this.user.bio} - Download ${this.user.username}'s projects on Modrinth`
      : `Download ${this.user.username}'s projects on Modrinth`

    return {
      title: this.user.username + ' - Modrinth',
      meta: [
        {
          hid: 'og:type',
          name: 'og:type',
          content: 'website',
        },
        {
          hid: 'og:title',
          name: 'og:title',
          content: this.user.username,
        },
        {
          hid: 'apple-mobile-web-app-title',
          name: 'apple-mobile-web-app-title',
          content: description,
        },
        {
          hid: 'og:description',
          name: 'og:description',
          content: description,
        },
        {
          hid: 'description',
          name: 'description',
          content: `${this.user.bio} - Download ${this.user.username}'s projects on Modrinth`,
        },
        {
          hid: 'og:image',
          name: 'og:image',
          content:
            this.user.avatar_url || 'https://cdn.modrinth.com/placeholder.png',
        },
      ],
    }
  },
  computed: {
    authUrl() {
      return `${process.env.authURLBase}auth/init?url=${process.env.domain}${this.$route.path}`
    },
    projectTypes() {
      const obj = {}

      for (const project of this.projects) {
        obj[project.project_type] = true
      }

      return Object.keys(obj)
    },
  },
  methods: {
    sumDownloads() {
      let sum = 0

      for (const projects of this.projects) {
        sum += projects.downloads
      }

      return this.$formatNumber(sum)
    },
    sumFollows() {
      let sum = 0

      for (const projects of this.projects) {
        sum += projects.followers
      }

      return this.$formatNumber(sum)
    },
    showPreviewImage(files) {
      const reader = new FileReader()
      this.icon = files[0]
      reader.readAsDataURL(this.icon)
      reader.onload = (event) => {
        this.previewImage = event.target.result
      }
    },
    async saveChanges() {
      this.$nuxt.$loading.start()
      try {
        if (this.icon) {
          await this.$axios.patch(
            `user/${this.$auth.user.id}/icon?ext=${
              this.icon.type.split('/')[this.icon.type.split('/').length - 1]
            }`,
            this.icon,
            this.$defaultHeaders()
          )
        }

        const data = {
          email: this.user.email,
          bio: this.user.bio,
        }
        if (this.user.username !== this.$auth.user.username) {
          data.username = this.user.username
        }

        await this.$axios.patch(
          `user/${this.$auth.user.id}`,
          data,
          this.$defaultHeaders()
        )
        await this.$store.dispatch('auth/fetchUser', {
          token: this.$auth.token,
        })

        this.isEditing = false
      } catch (err) {
        this.$notify({
          group: 'main',
          title: 'An error occurred',
          text: err.response.data.description,
          type: 'error',
        })
      }
      this.$nuxt.$loading.finish()
    },
    async cycleSearchDisplayMode() {
      const value = this.$cosmetics.searchDisplayMode.user
      const newValue = this.$cycleValue(value, this.$tag.projectViewModes)
      await this.$store.dispatch('cosmetics/saveSearchDisplayMode', {
        projectType: 'user',
        mode: newValue,
        $cookies: this.$cookies,
      })
    },
  },
}
</script>

<style lang="scss" scoped>
.user-header-wrapper {
  display: flex;
  margin: 0 auto -1.5rem;
  max-width: 80rem;

  .user-header {
    position: relative;
    z-index: 4;
    display: flex;
    width: 100%;
    padding: 0 1rem;
    gap: 1rem;
    align-items: center;

    .username {
      display: none;
      font-size: 2rem;
      margin-bottom: 2.5rem;
    }
  }
}

.mobile-username {
  margin: 0.25rem 0;
}

@media screen and (min-width: 501px) {
  .mobile-username {
    display: none;
  }

  .user-header-wrapper .user-header .username {
    display: block;
  }
}

.sidebar {
  padding-top: 2.5rem;
}

.sidebar__item:not(:last-child) {
  margin: 0 0 0.75rem 0;
}

.profile-picture {
  border-radius: var(--size-rounded-lg);
  height: 8rem;
  width: 8rem;
}

.username {
  font-size: var(--font-size-xl);
}

.bio {
  display: block;
  overflow-wrap: break-word;
}

.secondary-stat {
  align-items: center;
  display: flex;
  margin-bottom: 0.8rem;
}

.secondary-stat__icon {
  height: 1rem;
  width: 1rem;
}

.secondary-stat__text {
  margin-left: 0.4rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.date {
  cursor: default;
}

.github-button {
  display: inline-flex;
}

.inputs {
  margin-bottom: 1rem;

  input {
    margin-top: 0.5rem;
    width: 100%;
  }

  label {
    margin-bottom: 0;
  }
}

.button-group:first-child {
  margin-left: auto;
}

.textarea-wrapper {
  height: 10rem;
}
</style>
