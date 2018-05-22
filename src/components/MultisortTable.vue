<script>
import VDataTable from 'vuetify/src/components/VDataTable'
import VIcon from 'vuetify/src/components/VIcon'
import { consoleWarn } from 'vuetify/src/util/console'

// source: https://stackoverflow.com/questions/6913512/how-to-sort-an-array-of-objects-by-multiple-fields
const fieldSorter = (fields) => (a, b) => fields.map(o => {
  let dir = 1
  if (o[0] === '-') { dir = -1; o = o.substring(1) }
  return a[o] > b[o] ? dir : a[o] < b[o] ? -(dir) : 0
}).reduce((p, n) => p || n, 0)

export default {
  name: 'multisort-table',
  mixins: [VDataTable],
  data () {
    return {
      defaultPagination: {
        page: 1,
        rowsPerPage: 5,
        sortBy: {},
        totalItems: 0
      }
    }
  },
  created () {
    this.defaultPagination.sortBy = {}
    const firstSortable = this.headers.find(h => (
      !('sortable' in h) || h.sortable)
    )

    if (!this.disableInitialSort && firstSortable) {
      this.defaultPagination.sortBy[firstSortable.value] = true
    }

    this.initPagination()
  },
  methods: {
    sort (index, shiftKey) {
      var sortBy = this.computedPagination.sortBy
      sortBy = sortBy === index ? {} : sortBy

      for (var prop in sortBy) {
        if (sortBy.hasOwnProperty(prop)) {
          if ((prop !== index && !shiftKey) || sortBy[prop] === null) {
            delete sortBy[prop]
          }
        }
      }

      if (!sortBy.hasOwnProperty(index)) {
        sortBy[index] = true
      } else if (sortBy[index] === true) {
        sortBy[index] = false
      } else if (sortBy[index] === false) {
        delete sortBy[index]
      } else {
        sortBy[index] = true
      }

      this.updatePagination({ sortBy: sortBy })
    },

    genHeaderSortingData (header, children, data, classes) {
      if (!('value' in header)) {
        consoleWarn('Headers must have a value property that corresponds to a value in the v-model array', this)
      }

      data.attrs.tabIndex = 0
      data.on = {
        click: e => {
          this.expanded = {}
          this.sort(header.value, e.shiftKey)
        },
        keydown: e => {
          // check for space
          if (e.keyCode === 32) {
            e.preventDefault()
            this.sort(header.value)
          }
        }
      }

      classes.push('sortable')
      const icon = this.$createElement(VIcon, {
        props: {
          small: true
        }
      }, this.sortIcon)
      if (!header.align || header.align === 'left') {
        children.push(icon)
      } else {
        children.unshift(icon)
      }

      const pagination = this.computedPagination
      const beingSorted = pagination.sortBy.hasOwnProperty(header.value) && pagination.sortBy[header.value] !== null
      if (beingSorted) {
        classes.push('active')

        if (Object.keys(pagination.sortBy).length > 1) {
          children.push(this.$createElement('small', { attrs: { class: 'grey--text text--lighten-1' } }, Object.keys(pagination.sortBy).indexOf(header.value) + 1))
        }

        if (pagination.sortBy[header.value] !== true) {
          classes.push('desc')
          data.attrs['aria-sort'] = 'descending'
          data.attrs['aria-label'] += ': Sorted descending. Activate to remove sorting.' // TODO: Localization
        } else {
          classes.push('asc')
          data.attrs['aria-sort'] = 'ascending'
          data.attrs['aria-label'] += ': Sorted ascending. Activate to sort descending.' // TODO: Localization
        }
      } else {
        data.attrs['aria-label'] += ': Not sorted. Activate to sort ascending.' // TODO: Localization
      }
    }
  },
  props: {
    customSort: {
      type: Function,
      default: (items, sortBy, isDescending) => {
        if (sortBy === null) return items

        var sortInfo = []
        var keys = Object.keys(sortBy)
        for (var i = 0; i < keys.length; i++) {
          var index = keys[i]
          var s = ''
          if (sortBy[index] === null) {
            continue
          }

          if (sortBy[index] === false) {
            s += '-'
          }

          s += index
          sortInfo.push(s)
        }

        items = items.sort(fieldSorter(sortInfo))

        return items
      }
    }
  }
}
</script>

<style scoped lang="scss">
  small {
    position: absolute;
  }
</style>
