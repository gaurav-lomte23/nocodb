<script lang="ts" setup>
import type { ColumnType, LinkToAnotherRecordType } from 'nocodb-sdk'
import { RelationTypes, UITypes, isVirtualCol } from 'nocodb-sdk'
import { SwipeDirection, breakpointsTailwind } from '@vueuse/core'
import {
  DropZoneRef,
  computed,
  onKeyStroke,
  onMounted,
  provide,
  ref,
  useBreakpoints,
  usePointerSwipe,
  useSharedFormStoreOrThrow,
  useStepper,
} from '#imports'

enum TransitionDirection {
  Left = 'left',
  Right = 'right',
}

const { md } = useBreakpoints(breakpointsTailwind)

const { v$, formState, formColumns, submitForm, submitted, secondsRemain, sharedFormView, onReset } = useSharedFormStoreOrThrow()

const isTransitioning = ref(false)

const transitionName = ref<TransitionDirection>(TransitionDirection.Left)

const el = ref<HTMLDivElement>()

provide(DropZoneRef, el)

const steps = computed(() => {
  if (!formColumns.value) return []

  return formColumns.value.reduce<string[]>((acc, column) => {
    const title = column.label || column.title

    if (!title) return acc

    acc.push(title)

    return acc
  }, [])
})

const { index, goToPrevious, goToNext, isFirst, isLast, goTo } = useStepper(steps)

const field = computed(() => formColumns.value?.[index.value])

function isRequired(column: ColumnType, required = false) {
  let columnObj = column
  if (
    columnObj.uidt === UITypes.LinkToAnotherRecord &&
    columnObj.colOptions &&
    (columnObj.colOptions as { type: RelationTypes }).type === RelationTypes.BELONGS_TO
  ) {
    columnObj = formColumns.value?.find(
      (c) => c.id === (columnObj.colOptions as LinkToAnotherRecordType).fk_child_column_id,
    ) as ColumnType
  }

  return required || (columnObj && columnObj.rqd && !columnObj.cdf)
}

function transition(direction: TransitionDirection) {
  isTransitioning.value = true
  transitionName.value = direction

  setTimeout(() => {
    transitionName.value =
      transitionName.value === TransitionDirection.Left ? TransitionDirection.Right : TransitionDirection.Left
  }, 500)

  setTimeout(() => {
    isTransitioning.value = false

    setTimeout(focusInput, 100)
  }, 1000)
}

async function goNext() {
  if (isLast.value) return

  if (!field.value || !field.value.title) return

  const validationField = v$.value.localState[field.value.title]

  if (validationField) {
    const isValid = await validationField.$validate()
    if (!isValid) return
  }

  transition(TransitionDirection.Left)

  goToNext()
}

async function goPrevious() {
  if (isFirst.value) return

  transition(TransitionDirection.Right)

  goToPrevious()
}

function focusInput() {
  if (document && typeof document !== 'undefined') {
    const inputEl =
      (document.querySelector('.nc-cell input') as HTMLInputElement) ||
      (document.querySelector('.nc-cell textarea') as HTMLTextAreaElement)

    if (inputEl) {
      inputEl.select()
      inputEl.focus()
    }
  }
}

function resetForm() {
  v$.value.$reset()
  submitted.value = false
  transition(TransitionDirection.Right)
  goTo(steps.value[0])
}

onReset(resetForm)

onKeyStroke(['ArrowLeft', 'ArrowDown'], goPrevious)
onKeyStroke(['ArrowRight', 'ArrowUp', 'Enter', 'Space'], goNext)

onMounted(() => {
  focusInput()

  if (!md.value) {
    const { direction } = usePointerSwipe(el, {
      onSwipe: () => {
        if (isTransitioning.value) return

        if (direction.value === SwipeDirection.LEFT) {
          goNext()
        } else if (direction.value === SwipeDirection.RIGHT) {
          goPrevious()
        }
      },
    })
  }
})
</script>

<template>
  <div ref="el" class="pt-8 md:p-0 w-full h-full flex flex-col">
    <div
      v-if="sharedFormView"
      style="height: max(40vh, 250px); min-height: 250px"
      class="max-w-[max(33%,600px)] mx-auto flex flex-col justify-end"
    >
      <div class="px-4 md:px-0 flex flex-col justify-end">
        <h1 class="prose-2xl font-bold self-center my-4">{{ sharedFormView.heading }}</h1>

        <h2
          v-if="sharedFormView.subheading && sharedFormView.subheading !== ''"
          class="prose-lg text-slate-500 dark:text-slate-300 self-center mb-4 leading-6"
        >
          {{ sharedFormView?.subheading }}
        </h2>
      </div>
    </div>

    <div class="h-full w-full flex items-center px-4 md:px-0">
      <Transition :name="`slide-${transitionName}`" :duration="1000" mode="out-in">
        <div
          ref="el"
          :key="field.title"
          class="color-transition h-full flex flex-col mt-6 gap-4 w-full max-w-[max(33%,600px)] m-auto"
        >
          <div v-if="field && !submitted" class="flex flex-col gap-2">
            <div class="flex nc-form-column-label">
              <LazySmartsheetHeaderVirtualCell
                v-if="isVirtualCol(field)"
                :column="{ ...field, title: field.label || field.title }"
                :required="isRequired(field, field.required)"
                :hide-menu="true"
              />

              <LazySmartsheetHeaderCell
                v-else
                :column="{ ...field, title: field.label || field.title }"
                :required="isRequired(field, field.required)"
                :hide-menu="true"
              />
            </div>

            <div>
              <LazySmartsheetVirtualCell
                v-if="isVirtualCol(field)"
                class="mt-0 nc-input"
                :class="`nc-form-input-${field.title.replaceAll(' ', '')}`"
                :column="field"
              />

              <LazySmartsheetCell
                v-else
                v-model="formState[field.title]"
                class="nc-input"
                :class="`nc-form-input-${field.title.replaceAll(' ', '')}`"
                :column="field"
                :edit-enabled="true"
              />

              <div class="flex flex-col gap-2 text-slate-500 dark:text-slate-300 text-[0.75rem] my-2 px-1">
                <div v-for="error of v$.localState[field.title]?.$errors" :key="error" class="text-red-500">
                  {{ error.$message }}
                </div>

                <div class="block text-[14px]">
                  {{ field.description }}
                </div>
              </div>
            </div>
          </div>

          <div class="ml-1 mt-4 flex w-full text-lg">
            <div class="flex-1 flex justify-center">
              <div v-if="isLast && !submitted && !v$.$invalid" class="text-center my-4">
                <button type="submit" class="uppercase scaling-btn prose-sm" @click="submitForm">
                  {{ $t('general.submit') }}
                </button>
              </div>

              <div v-else-if="!submitted" class="flex items-center gap-3">
                <a-tooltip
                  :title="v$.localState[field.title]?.$error ? v$.localState[field.title].$errors[0].$message : 'Go to next'"
                  :mouse-enter-delay="0.25"
                  :mouse-leave-delay="0"
                >
                  <button
                    class="bg-opacity-100 scaling-btn flex items-center gap-1"
                    :class="v$.localState[field.title]?.$error ? 'after:!bg-gray-100 after:!ring-red-500' : ''"
                    @click="goNext"
                  >
                    <Transition name="fade">
                      <span v-if="!v$.localState[field.title]?.$error" class="uppercase text-white">Ok</span>
                    </Transition>

                    <Transition name="slide-right" mode="out-in">
                      <MdiCloseCircleOutline v-if="v$.localState[field.title]?.$error" class="text-red-500 md:text-md" />
                      <MdiCheck v-else class="text-white md:text-md" />
                    </Transition>
                  </button>
                </a-tooltip>

                <!-- todo: i18n -->
                <div class="hidden md:flex text-sm text-gray-500 items-center gap-1">
                  Press Enter <MaterialSymbolsKeyboardReturn class="mt-1" />
                </div>
              </div>
            </div>
          </div>

          <Transition name="slide-left">
            <div v-if="submitted" class="flex flex-col justify-center items-center text-center">
              <div class="text-lg px-6 py-3 bg-green-300 text-gray-700 rounded">
                <template v-if="sharedFormView?.success_msg">
                  {{ sharedFormView?.success_msg }}
                </template>

                <template v-else>
                  <div class="flex flex-col gap-1">
                    <div>Thank you!</div>

                    <div>You have successfully submitted the form data.</div>
                  </div>
                </template>
              </div>

              <div v-if="sharedFormView" class="mt-3">
                <p v-if="sharedFormView?.show_blank_form" class="text-xs text-slate-500 dark:text-slate-300 text-center my-4">
                  New form will be loaded after {{ secondsRemain }} seconds
                </p>

                <div v-if="sharedFormView?.submit_another_form" class="text-center">
                  <button type="button" class="scaling-btn bg-opacity-100" @click="resetForm">Submit Another Form</button>
                </div>
              </div>
            </div>
          </Transition>
        </div>
      </Transition>
    </div>

    <template v-if="!submitted">
      <div class="mb-24 md:my-4 select-none text-center text-gray-500 dark:text-slate-200">
        {{ index + 1 }} / {{ formColumns?.length }}
      </div>
    </template>

    <div class="relative flex w-full items-end">
      <Transition name="fade">
        <div
          v-if="!submitted"
          class="color-transition shadow-sm absolute bottom-18 right-1/2 transform translate-x-[50%] md:bottom-4 md:(right-12 transform-none) flex items-center bg-white border dark:bg-slate-500 rounded divide-x-1"
        >
          <a-tooltip :title="isFirst ? '' : 'Go to previous'" :mouse-enter-delay="0.25" :mouse-leave-delay="0">
            <button class="p-0.5 flex items-center group color-transition" @click="goPrevious">
              <MdiChevronLeft :class="isFirst ? 'text-gray-300' : 'group-hover:text-accent'" class="text-2xl md:text-md" />
            </button>
          </a-tooltip>

          <a-tooltip
            :title="v$.localState[field.title]?.$error ? '' : 'Go to next'"
            :mouse-enter-delay="0.25"
            :mouse-leave-delay="0"
          >
            <button class="p-0.5 flex items-center group color-transition" @click="goNext">
              <MdiChevronRight
                :class="isLast || v$.localState[field.title]?.$error ? 'text-gray-300' : 'group-hover:text-accent'"
                class="text-2xl md:text-md"
              />
            </button>
          </a-tooltip>
        </div>
      </Transition>

      <GeneralPoweredBy />
    </div>
  </div>
</template>

<style lang="scss">
:global(html, body) {
  @apply overscroll-x-none;
}

.nc-form-column-label {
  > * {
    @apply !prose-lg;
  }

  .nc-icon {
    @apply mr-2;
  }
}
</style>
