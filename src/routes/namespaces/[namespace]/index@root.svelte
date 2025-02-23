<script context="module" lang="ts">
  import type { Load } from '@sveltejs/kit';
  import type { DescribeNamespaceResponse } from '$types';

  export const load: Load = async function ({ params, url, stuff }) {
    const { searchParams } = url;

    if (searchParams.has('time-range')) searchParams.delete('time-range');

    const namespace = params.namespace;
    const namespaces: DescribeNamespaceResponse[] = stuff.namespaces;

    const currentNamespace = namespaces.find(
      (namespaceConfig) => namespaceConfig.namespaceInfo.name === namespace,
    );

    if (!currentNamespace) {
      return {
        error: `The namespace "${namespace}" does not exist.`,
        status: 404,
      };
    }

    const clusters = getClusters(currentNamespace);

    return {
      props: {
        currentNamespace,
        clusters,
      },
    };
  };
</script>

<script lang="ts">
  import { onMount } from 'svelte';
  import { dev } from '$app/env';
  import { page } from '$app/stores';

  import { temporalVersion, uiVersion } from '$lib/stores/versions';
  import { supportsReverseOrder } from '$lib/stores/event-view';
  import { lastUsedNamespace } from '$lib/stores/namespaces';
  import { searchAttributes } from '$lib/stores/search-attributes';

  import { fromSecondsToDaysOrHours } from '$lib/utilities/format-date';
  import { getClusters } from '$lib/utilities/get-clusters';

  import PageTitle from '$lib/holocene/page-title.svelte';
  import Table from '$lib/holocene/table/table.svelte';
  import TableHeaderRow from '$lib/holocene/table/table-header-row.svelte';
  import TableRow from '$lib/holocene/table/table-row.svelte';
  import { getEnabledCategories } from 'trace_events';

  export let currentNamespace: DescribeNamespaceResponse;
  export let clusters: string;

  onMount(() => {
    $lastUsedNamespace = currentNamespace?.namespaceInfo?.name;
  });
</script>

<PageTitle
  title={`Namespaces | ${currentNamespace?.namespaceInfo?.name}`}
  url={$page.url.href}
/>
<h2 class="text-2xl" data-cy="namespace-title">
  Namespace: {currentNamespace?.namespaceInfo?.name}
</h2>
<div class="flex">
  <article class="namespace-info w-full p-4">
    <h1 class="my-4 text-lg font-medium">Details</h1>
    <p data-cy="namespace-description">
      <span class="mr-2 font-medium">Description:</span>
      {currentNamespace?.namespaceInfo?.description}
    </p>
    <p data-cy="namespace-owner">
      <span class="mr-2 font-medium">Owner:</span>
      {currentNamespace?.namespaceInfo?.ownerEmail || 'Unknown'}
    </p>
    <p data-cy="namespace-global">
      <span class="mr-2 font-medium">Global?</span>
      {currentNamespace?.isGlobalNamespace ? 'Yes' : 'No'}
    </p>
    <p data-cy="namespace-retention">
      <span class="mr-2 font-medium">Retention Period:</span>
      {fromSecondsToDaysOrHours(
        currentNamespace?.config?.workflowExecutionRetentionTtl.toString(),
      )}
    </p>
    <p data-cy="namespace-history">
      <span class="mr-2 font-medium">History Archival:</span>
      {currentNamespace?.config?.historyArchivalState}
    </p>
    <p data-cy="namespace-visibility">
      <span class="mr-2 font-medium">Visibility Archival:</span>
      {currentNamespace?.config?.visibilityArchivalState}
    </p>
    <p data-cy="namespace-failover">
      <span class="mr-2 font-medium">Failover Version:</span>
      {currentNamespace?.failoverVersion}
    </p>
    <p data-cy="namespace-clusters">
      <span class="mr-2 font-medium">Clusters:</span>
      {clusters}
    </p>
  </article>

  <article class="namespace-info w-full p-4">
    <h1 class="my-4 text-lg font-medium">Versions</h1>
    <p data-cy="server-version">
      <span class="mr-2 font-medium">Temporal Server Version:</span>
      {$temporalVersion}
    </p>
    <p data-cy="ui-version">
      <span class="mr-2 font-medium">Temporal UI Version:</span>
      {$uiVersion}
    </p>
    {#if dev}
      <p data-cy="supports-reverse-order">
        <span class="mr-2 font-medium">Supports Descending Event History:</span>
        {$supportsReverseOrder}
      </p>
    {/if}
  </article>
</div>

{#if $searchAttributes}
  <section>
    <h1 class="my-4 text-lg font-medium">Search Attributes</h1>
    <Table class="w-full">
      <TableHeaderRow slot="headers">
        <th>Key</th>
        <th>Type</th>
      </TableHeaderRow>
      {#each Object.entries($searchAttributes) as [key, type]}
        <TableRow>
          <td>{key}</td>
          <td>{type}</td>
        </TableRow>
      {/each}
    </Table>
  </section>
{/if}
