<script lang="ts">
import { Buffer } from 'buffer';
import { onDestroy, onMount } from 'svelte';
import type { Unsubscriber } from 'svelte/store';

import PreferencesConnectionCreationRendering from '/@/lib/preferences/PreferencesConnectionCreationOrEditRendering.svelte';
import DetailsPage from '/@/lib/ui/DetailsPage.svelte';
import WarningMessage from '/@/lib/ui/WarningMessage.svelte';
import type { IConfigurationPropertyRecordedSchema } from '/@api/configuration/models.js';
import type {
  ProviderContainerConnectionInfo,
  ProviderInfo,
  ProviderKubernetesConnectionInfo,
} from '/@api/provider-info';

import { providerInfos } from '../../stores/providers';
import IconImage from '../appearance/IconImage.svelte';
import { isContainerConnection } from './Util';

export let properties: IConfigurationPropertyRecordedSchema[] = [];
export let providerInternalId: string | undefined = undefined;
export let name: string | undefined = undefined;

let connectionInfo: ProviderContainerConnectionInfo | ProviderKubernetesConnectionInfo;
let scope: string;
let providerInfo: ProviderInfo;

let providersUnsubscribe: Unsubscriber;
onMount(async () => {
  providersUnsubscribe = providerInfos.subscribe(providerInfosValue => {
    const providers = providerInfosValue;
    const connectionName = Buffer.from(name ?? '', 'base64').toString();
    providerInfo = providers.filter(provider => provider.internalId === providerInternalId)[0];
    connectionInfo = providerInfo.containerConnections.filter(connection => connection.name === connectionName)[0];
  });
  scope = isContainerConnection(connectionInfo) ? 'ContainerConnection' : 'KubernetesConnection';
});

onDestroy(() => {
  if (providersUnsubscribe) {
    providersUnsubscribe();
  }
});

async function editConnection(
  internalProviderId: string,
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  params: { [key: string]: any },
  key: symbol,
  keyLogger: (key: symbol, eventName: 'log' | 'warn' | 'error' | 'finish', args: string[]) => void,
  tokenId: number | undefined,
  taskId: number | undefined,
): Promise<void> {
  await window.editProviderConnectionLifecycle(
    internalProviderId,
    connectionInfo,
    params,
    key,
    keyLogger,
    tokenId,
    taskId,
  );
}
</script>

{#if providerInfo && connectionInfo}
  <DetailsPage title={connectionInfo.name}>
    {#snippet contentSnippet()}
      <div class="text-[var(--pd-content-text)]">
        <PreferencesConnectionCreationRendering
          providerInfo={providerInfo}
          connectionInfo={connectionInfo}
          properties={properties}
          propertyScope={scope}
          callback={editConnection} />
      </div>
    {/snippet}
    {#snippet iconSnippet()}
      <IconImage image={providerInfo?.images?.icon} alt={providerInfo?.name} class="max-h-10" />
    {/snippet}
    {#snippet subtitleSnippet()}
      {#if connectionInfo.status === 'started'}
        <WarningMessage
          error="This may restart the container or Kubernetes engine. Existing containers or pods may be stopped." />
      {/if}
    {/snippet}
  </DetailsPage>
{/if}
