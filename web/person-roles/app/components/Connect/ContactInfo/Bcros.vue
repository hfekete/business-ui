<template>
  <div class="space-y-5">
    <ul class="space-y-1">
      <li>
        <span class="inline-flex items-center gap-1">
          <UIcon
            name="i-mdi-phone"
            class="mr-2 size-4 text-blue-350"
          />
          {{ $t('contactInfo.bcros.tollFree.title') }}
          <a
            class="text-blue-500 underline"
            href="tel:+1-877-526-1526"
          >
            1-877-526-1526
          </a>
        </span>
      </li>
      <li>
        <span class="inline-flex items-center gap-1">
          <UIcon
            name="i-mdi-phone"
            class="mr-2 size-4 text-blue-350"
          />
          {{ $t('contactInfo.bcros.victoriaOffice.title') }}
          <a
            class="text-blue-500 underline"
            href="tel:+1-250-387-7848"
          >
            250-387-7848
          </a>
        </span>
      </li>
      <li>
        <span class="inline-flex items-center gap-1">
          <UIcon
            name="i-mdi-email"
            class="mr-2 size-4 text-blue-350"
          />
          {{ $t('contactInfo.bcros.email.title') }}
          <a
            class="text-blue-500 underline"
            href="mailto:bcregistries@gov.bc.ca?subject=BC%20Registries%20Support%20Request"
            target="_blank"
          >
            BCRegistries@gov.bc.ca
          </a>
        </span>
      </li>
    </ul>

    <p>
      <strong>{{ $t('contactInfo.bcros.hours.title') }}</strong>
      <br>
      {{ $t('contactInfo.bcros.hours.value') }}
    </p>
  </div>
</template>
