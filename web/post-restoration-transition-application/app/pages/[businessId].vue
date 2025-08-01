<script setup lang="ts">
import { h, resolveComponent } from 'vue'
import { fromIsoToUsDateFormat } from '~/utils/uidate'
import { areUiAddressesEqual, areApiAddressesEqual } from '~/utils/address'

const t = useNuxtApp().$i18n.t
const rtc = useRuntimeConfig().public
const preexistingCompanyProvisions = rtc.preexistingCompanyProvisions as string

useHead({
  title: t('transitionApplication.title')
})

const route = useRoute()

definePageMeta({
  layout: 'form',
  middleware: () => {
    // redirect to reg home with return url if user unauthenticated
    const { $keycloak, $config } = useNuxtApp()
    if (!$keycloak.authenticated) {
      const returnUrl = encodeURIComponent(window.location.href)
      return navigateTo(
        `${$config.public.registryHomeUrl}login?return=${returnUrl}`,
        { external: true }
      )
    }
  }
})

const ConnectAddressDisplay = resolveComponent('ConnectAddressDisplay')

const businessId = route.params.businessId as string

const filingStore = usePostRestorationTransitionApplicationStore()
filingStore.init(businessId)
const {
  activeBusiness,
  articles,
  certifiedByLegalName,
  compPartyEmail,
  courtOrderNumber,
  directors,
  folio,
  isStaffOrSbcStaff,
  legalName,
  offices,
  planOfArrangement,
  regOfficeEmail
} = storeToRefs(filingStore)

const sectionErrors = ref({
  reviewAndConfirm: false,
  certify: false
})

// Define table columns
const officesColumns = ref([
  {
    accessorKey: 'officeType',
    header: t('label.office'),
    cell: ({ row }) => {
      return h(
        'div',
        { class: 'text-left font-bold text-bgGovColor-midGray' },
        t(`label.${row.original.officeType}`)
      )
    }
  },
  {
    accessorKey: 'mailingAddress',
    header: t('label.mailingAddress'),
    cell: ({ row }) => {
      return h(ConnectAddressDisplay, { address: row.original.mailingAddress })
    }
  },
  {
    accessorKey: 'deliveryAddress',
    header: t('label.deliveryAddress'),
    cell: ({ row }) => {
      if (areUiAddressesEqual(row.original.deliveryAddress, row.original.mailingAddress)) {
        return t('label.sameAsMailingAddress')
      }
      return h(ConnectAddressDisplay, { address: row.original.deliveryAddress })
    }
  }
])
const directorsColumns = ref([
  {
    accessorKey: 'officer',
    header: t('label.name'),
    cell: ({ row }) => {
      return h(
        'div',
        { class: 'text-left font-bold text-bgGovColor-midGray' },
        `${row.original.officer.firstName} ${row.original.officer.lastName}`
      )
    }
  },
  {
    accessorKey: 'mailingAddress',
    header: t('label.mailingAddress'),
    cell: ({ row }) => {
      return h(ConnectAddressDisplay, { address: formatAddressUi(row.original.mailingAddress) })
    }
  },
  {
    accessorKey: 'deliveryAddress',
    header: t('label.deliveryAddress'),
    cell: ({ row }) => {
      if (areApiAddressesEqual(row.original.deliveryAddress, row.original.mailingAddress)) {
        return t('label.sameAsMailingAddress')
      }
      return h(ConnectAddressDisplay, { address: formatAddressUi(row.original.deliveryAddress) })
    }
  },
  {
    accessorKey: 'startDate',
    header: t('label.effectiveDates'),
    cell: ({ row }) => {
      const directorRole = row.original.roles.find((role: Role) => role.roleType === 'Director')
      const fromDate = fromIsoToUsDateFormat(directorRole.appointmentDate)
      const toDate = fromIsoToUsDateFormat(directorRole.cessationDate) || t('label.current')
      return h(
        'div',
        { class: 'text-left text-wrap' },
        `${fromDate} ${t('label.to')} ${toDate}`
      )
    }
  },
  {
    accessorKey: 'action',
    header: '',
    cell: () => {
      return h(
        'div',
        { class: 'text-left font-bold text-bgGovColor-midGray' },
        'TODO: ADD CHANGE LINK'
      )
    }
  }
])

const displayLegalName = computed(() => {
  return legalName.value || t('text.legalNameCertifyPlaceHolder')
})

const showDateInputBox = ref(false)
const showPreviousResolutions = ref(false)
const showPreviousResolutionsDateLabel = computed(() => {
  return showPreviousResolutions.value
    ? t('text.previousResolutionsHide')
    : t('text.previousResolutionsShow')
})

const removeDateHandler = () => {
  articles.value.currentDate = undefined
}

const minArticleResolutionDate = computed(() => {
  return activeBusiness?.value?.foundingDate ? new Date(activeBusiness?.value?.foundingDate).toISOString() : ''
})
</script>

<template>
  <!-- NB: this is temporary so that tw pulls in needed classes for extended comps -->
  <!-- z-10 lg:shadow-sm bg-midnightBlue-900 divide-bcGovGray-300 justify-between -->
  <!-- border-b border-t border-bcGovGray-300 border-gray-300 ml-[5px] -->
  <div class="py-10 space-y-10">
    <section>
      <h1 class="mb-2">
        {{ $t('transitionApplication.title') }}
      </h1>
      <i18n-t
        keypath="text.transitionYourBusiness"
        tag="p"
        class="mb-4 text-[18px]"
      >
        <template #businessCorporationsAct>
          <em>{{ $t('label.businessCorporationsAct') }}</em>
        </template>
      </i18n-t>
    </section>
    <FormSection
      :title="$t('transitionApplication.subtitle.reviewAndConfirm')"
      :description="$t('text.reviewAndConfirmDescription')"
      :has-errors="sectionErrors.reviewAndConfirm"
      class="space-y-4"
    >
      <div class="space-y-6">
        <FormSubSection
          icon="i-mdi-domain"
          :title="$t('label.officeAddresses')"
          class="pb-6"
        >
          <FormDataList
            :data="offices"
            :columns="officesColumns"
            class="m-6"
            :ui="{
              th: 'text-[14px] text-left text-bcGovColor-midGray p-4',
              td: 'border-t-1 border-bcGovColor-hairlinesOnWhite text-[14px] whitespace-nowrap p-4 text-gray-500'
            }"
          />
          <InfoBox
            icon="i-mdi-information-outline"
            :title="$t('text.needChange')"
            :text="$t('text.goToMainFileAddressChange')"
            class="pl-6 pr-6 text-[14px]"
            title-class="text-[14px]"
          />
        </FormSubSection>

        <FormSubSection
          icon="i-mdi-account-multiple-plus"
          :title="$t('label.currentDirectors')"
          class="pb-6"
        >
          <FormDataList
            :data="directors"
            :columns="directorsColumns"
            class="m-6"
            :ui="{
              th: 'text-[14px] text-left text-bcGovColor-midGray p-4',
              td: 'border-t-1 border-bcGovColor-hairlinesOnWhite text-[14px] whitespace-nowrap p-4 text-gray-500'
            }"
          />
          <InfoBox
            icon="i-mdi-information-outline"
            :title="$t('text.needOtherChange')"
            :text="$t('text.deleteAndFileDirectorChange')"
            class="pl-6 pr-6 text-[14px]"
            title-class="text-[14px]"
          />
        </FormSubSection>

        <FormSubSection
          icon="i-mdi-handshake"
          :title="$t('label.articles')"
        >
          <ConnectFormSection
            :title="$t('label.resolutionOrCourtOrderDate')"
            class="text-base p-6 pb-2"
          >
            <template #title="{ error }">
              <p class="mb-2 sm:mb-0 sm:font-bold" :class="error ? 'text-red-600' : ''">
                {{ $t('label.resolutionOr') }}<br>
                {{ $t('label.courtOrderDate') }}
              </p>
            </template>
            <template #default>
              <div class="flex flex-col space-y-4">
                <p>{{ $t('text.articlesDescription') }}</p>
                <div>
                  <div v-if="articles?.currentDate" class="flex flex-row space-x-6">
                    <p>{{ fromIsoToUsDateFormat(articles?.currentDate) }}</p>
                    <UButton
                      icon="i-mdi-delete"
                      :label="$t('label.remove')"
                      :padded="false"
                      variant="ghost"
                      class="rounded text-base p-0 gap-1"
                      @click="removeDateHandler"
                    />
                  </div>
                  <div v-else-if="showDateInputBox">
                    <FormDateInputWithButtons
                      id="articlesCurrentDate"
                      v-model="articles.currentDate"
                      name="articles-current-date"
                      :label="$t('text.articlesDate')"
                      :min-date="minArticleResolutionDate"
                      :max-date="(new Date()).toISOString()"
                      @save="showDateInputBox=false"
                      @cancel="showDateInputBox=false"
                    />
                  </div>
                  <div v-else>
                    <UButton
                      icon="i-mdi-add"
                      :label="$t('label.addADate')"
                      :padded="false"
                      variant="outline"
                      class="rounded text-base pt-[11px] pb-[11px]"
                      @click="showDateInputBox=true"
                    />
                  </div>
                </div>
                <div>
                  <!-- todo: when validations come in, add error info box here -->
                </div>
                <div v-if="articles?.resolutionDates?.length">
                  <UButton
                    :label="showPreviousResolutionsDateLabel"
                    variant="link"
                    class="pr-0 pl-0 text-base"
                    @click="showPreviousResolutions=!showPreviousResolutions"
                  />
                  <div v-if="showPreviousResolutions">
                    <p
                      v-for="(resolutionDate, i) in articles?.resolutionDates"
                      :key="i"
                    >
                      {{ resolutionDate }}
                    </p>
                  </div>
                </div>
              </div>
            </template>
          </ConnectFormSection>
        </FormSubSection>
      </div>
    </FormSection>
    <FormSection
      :title="$t('transitionApplication.subtitle.documentDelivery')"
      :description="$t('text.documentDelivery')"
      :has-errors="sectionErrors.reviewAndConfirm"
      class="space-y-4"
    >
      <FormSubSection
        title=""
        class="space-y-6 p-6"
      >
        <ConnectFormSection :title="$t('label.registeredOffice')">
          {{ regOfficeEmail }}
        </ConnectFormSection>
        <ConnectFormSection :title="$t('label.completingParty')">
          <ConnectFormInput
            v-model="compPartyEmail"
            :name="'documentDelivery.completingPartyEmail'"
            :label="$t('label.emailAddressOptional')"
          />
        </ConnectFormSection>
      </FormSubSection>
    </FormSection>
    <FormSection
      :title="$t('transitionApplication.subtitle.courtOrder')"
      :description="$t('text.courtOrder')"
      :has-errors="sectionErrors.reviewAndConfirm"
      class="space-y-4"
    >
      <FormSubSection
        title=""
        class="space-y-6 p-6"
      >
        <ConnectFormSection :title="$t('label.courtOrderNumber')">
          <ConnectFormInput
            v-model="courtOrderNumber"
            :name="'courtOrder.number'"
            :label="$t('label.courtOrderNumberOptional')"
          />
        </ConnectFormSection>
        <ConnectFormSection :title="$t('label.planOfArrangement')">
          <UCheckbox
            v-model="planOfArrangement"
            :label="$t('label.planOfArrangement')"
            :ui="{ base: 'cursor-pointer mt-1', label: 'cursor-pointer', wrapper: 'w-fit' }"
          />
        </ConnectFormSection>
      </FormSubSection>
    </FormSection>
    <FormSection
      :title="$t('transitionApplication.subtitle.folio')"
      :description="$t('text.folioOrReferenceNumber')"
      :has-errors="sectionErrors.reviewAndConfirm"
      class="space-y-4"
    >
      <FormSubSection
        title=""
        class="space-y-6 p-6"
      >
        <ConnectFormSection :title="$t('label.folioOrReferenceNumber')">
          <ConnectFormInput
            v-model="folio"
            :name="'business.folio'"
            :label="$t('label.folioOrReferenceNumberOptional')"
          />
        </ConnectFormSection>
      </FormSubSection>
    </FormSection>
    <FormSection
      :title="$t('transitionApplication.subtitle.companyProvisions')"
      :has-errors="sectionErrors.reviewAndConfirm"
    >
      <FormSubSection
        title=""
        class="p-6"
      >
        <p class="font-bold">
          {{ $t('text.companyProvisionsHeading') }}
        </p>
        <i18n-t
          keypath="text.companyProvisionsText"
          tag="p"
          class="mb-4"
        >
          <template #businessCorporationsAct>
            <em>{{ $t('label.businessCorporationsAct') }}</em>
          </template>
        </i18n-t>
        <ULink
          :to="preexistingCompanyProvisions"
          external
          target="_blank"
          active-class="text-bcGovColor-activeBlue"
          inactive-class="text-bcGovColor-activeBlue"
        >
          {{ $t('text.companyProvisionsURL') }}
        </ULink>
      </FormSubSection>
    </FormSection>
    <FormSection
      :title="$t('transitionApplication.subtitle.certify')"
      :description="$t('text.certifySectionDescription')"
      :has-errors="sectionErrors.certify"
      class="space-y-4"
    >
      <FormSubSection
        title=""
        class="space-y-6 p-6"
      >
        <ConnectFormSection
          :title="$t('label.legalName')"
        >
          <ConnectFormInput
            v-model="legalName"
            :name="'documentDelivery.completingPartyEmail'"
            :label="$t('text.legalNameOfAuthorizedPerson')"
            :placeholder="$t('text.legalNameOfAuthorizedPerson')"
          />
        </ConnectFormSection>
        <ConnectFormSection title=" ">
          <FormCertify
            v-model="certifiedByLegalName"
            :legal-name="displayLegalName || $t('text.legalNameCertifyPlaceHolder')"
          >
            <template v-if="isStaffOrSbcStaff" #certifyText>
              <i18n-t
                keypath="text.certifiesItHasRelevantKnowledgeStaff"
                tag="p"
                class="mt-[-2px]"
              >
                <template #legalName>
                  <strong>{{ displayLegalName }}</strong>
                </template>
              </i18n-t>
            </template>
          </FormCertify>
        </ConnectFormSection>
      </FormSubSection>
    </FormSection>
    <FormSection
      :title="$t('transitionApplication.subtitle.shares')"
      :has-errors="sectionErrors.reviewAndConfirm"
    >
      <FormSubSection
        :title="$t('text.sharesTitle')"
        icon="i-mdi-sitemap"
        class="space-y-6 p-6"
      >
        <Shares />
      </FormSubSection>
    </FormSection>
    <!-- NB: needed so that the tw classes are loaded -->
    <!-- <main class="app-inner-container app-body">
      <div class="flex flex-col lg:flex-row lg:gap-6 grow">
        <div class="grow max-w-full overflow-hidden">
          <slot />
        </div>

        <div class="shrink-0 lg:w-[300px] lg:static sticky lg:mt-10">
          <ConnectFeeWidget class="sticky lg:top-10" />
        </div>
      </div>
    </main> -->
  </div>
</template>
