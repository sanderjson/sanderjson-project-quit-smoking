<script>
  import { onMount } from "svelte";
  import {
    errMessage,
    API_ENDPOINT,
    indexActiveHabit,
    userHabitsActive,
    userProfile,
    isLSDataOutdated
  } from "../stores.js";
  import { push } from "svelte-spa-router";
  import AppHeader from "../components/AppHeader.svelte";
  import TwentyTwentyOne from "../svg/2021.svelte";
  import AppModal from "../components/AppModal.svelte";
  import FormHabitAddEditDelete from "../components/FormHabitAddEditDelete.svelte";
  import ContentWrapper from "../components/ContentWrapper.svelte";
  import AppHeaderLocalScore from "../components/AppHeaderLocalScore.svelte";
  import AppHeaderLocalTitle from "../components/AppHeaderLocalTitle.svelte";

  let tempLocalUserHabit = $userHabitsActive[$indexActiveHabit];

  let contentModalDelete = {
    title: "Are You Sure You Want to Delete?",
    details: "You will lose all data associated with this habit.",
    button: "Delete Habit Data"
  };

  const handleModalDeleteAction = async () => {
    const fetchURL =
      $API_ENDPOINT + `/habits/${tempLocalUserHabit.adminIdHabit}`;
    const fetchOptions = {
      method: "DELETE",
      headers: {
        "content-type": "application/json"
      },

      body: JSON.stringify({
        ...tempLocalUserHabit
      })
    };

    const handleErrors = res => {
      if (!res.ok) {
        return res.text().then(text => {
          throw text;
        });
      }
      return res.json();
    };

    const postData = await fetch(fetchURL, fetchOptions)
      .then(handleErrors)
      .then(res => {
        console.log("res", res);
        let newHabitData = $userHabitsActive;
        newHabitData[$indexActiveHabit] = {};
        userHabitsActive.set(newHabitData);
        userProfile.set(res.userProfile);
        isLSDataOutdated.set(true);
      })
      .catch(err => {
        // console.clear();
        errMessage.set(err);
        push(`/error`);
      });

    habitDeleteWarning = false;
  };

  let habitDeleteWarning = false;
  const handleDelete = () => {
    habitDeleteWarning = true;
  };

  const handleSubmitEditExistingHabit = async () => {
    const fetchURL =
      $API_ENDPOINT + `/habits/${tempLocalUserHabit.adminIdHabit}`;
    const fetchOptions = {
      method: "PATCH",
      headers: {
        "content-type": "application/json"
      },

      body: JSON.stringify({
        ...tempLocalUserHabit
      })
    };

    const handleErrors = res => {
      if (!res.ok) {
        return res.text().then(text => {
          throw text;
        });
      }
      return res.json();
    };

    const postData = await fetch(fetchURL, fetchOptions)
      .then(handleErrors)
      .then(res => {
        // console.log("res", res);
        let newHabitData = $userHabitsActive;
        newHabitData[$indexActiveHabit] = res.updatedHabit;
        userHabitsActive.set(newHabitData);
        isLSDataOutdated.set(true);
      })
      .catch(err => {
        // console.clear();
        errMessage.set(err);
        push(`/error`);
      });
  };
</script>

<!-- <AppHeader>
  <TwentyTwentyOne />
</AppHeader> -->

<ContentWrapper>
  <div>
    <AppHeaderLocalScore />
    <AppHeaderLocalTitle
      title={'Edit Habit'}
      subtitle={'Fill out this form to edit'} />
    <div class="mt-6">
      <FormHabitAddEditDelete
        {tempLocalUserHabit}
        actionTitle="Update Habit"
        handleSubmit={handleSubmitEditExistingHabit}
        altActionTitle="Delete"
        handleAltAction={handleDelete} />
    </div>
  </div>
</ContentWrapper>

{#if habitDeleteWarning}
  <AppModal contentModal={contentModalDelete}>
    <button
      on:click={handleModalDeleteAction}
      type="button"
      class="inline-flex justify-center w-full rounded-md border
      border-transparent shadow-sm px-4 py-2 bg-blue-900 text-base font-medium
      text-white hover:bg-blue-500 focus:outline-none focus:ring-2
      focus:ring-offset-2 focus:ring-blue-700 sm:text-sm">
      {contentModalDelete.button}
    </button>
  </AppModal>
{/if}
