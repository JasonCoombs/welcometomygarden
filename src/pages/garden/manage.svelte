<script>
  import { _ } from 'svelte-i18n';
  import { goto, redirect } from '@sveltech/routify';
  import { updateGardenLocally } from '@/stores/garden';
  import { user } from '@/stores/auth';
  import notify from '@/stores/notification';
  import { Progress } from '@/components/UI';
  import { updateGarden } from '@/api/garden';
  import Form from '@/components/Garden/Form.svelte';
  import routes from '@/routes';

  if (!$user || !$user.emailVerified) {
    notify.warning('Please verify your email first.', 8000);
    $redirect(routes.ACCOUNT);
  }
  let updatingGarden = false;

  const submit = async (e) => {
    const garden = e.detail;
    updatingGarden = true;
    try {
      const newGarden = await updateGarden({
        ...garden,
        photo: garden.photo && garden.photo.files ? garden.photo.files[0] : null
      });
      updatingGarden = false;
      await updateGardenLocally(newGarden);

      let notifyMsg;
      newGarden.photo
        ? (notifyMsg = $_('garden.notify.update') + ' ' + $_('garden.notify.photo'))
        : (notifyMsg = $_('garden.notify.update'));
      notify.success(notifyMsg, 10000);
      $goto(`${routes.MAP}/garden/${$user.id}`);
    } catch (ex) {
      console.log(ex);
      updatingGarden = false;
    }
  };
</script>

<svelte:head>
  <title>{$_('garden.manage.title')} | Welcome To My Garden</title>
</svelte:head>

<Progress active={updatingGarden} />

<Form on:submit={submit} isUpdate isSubmitting={updatingGarden} garden={$user.garden} />
