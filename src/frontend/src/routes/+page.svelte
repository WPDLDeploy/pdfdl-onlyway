<script>
  let download_images = false;
  let download_as_pdf = false; // 0 = epub, 1 = pdf
  let is_paid_story = false;
  let invalid_url = false;
  let after_download_page = false;
  let credentials = {
    username: "",
    password: "",
  };
  let download_id = "";
  let mode = "";
  let input_url = "";

  let button_disabled = false;
  $: button_disabled =
    !input_url ||
    (is_paid_story && !(credentials.username && credentials.password));

  $: url =
    `/download/` +
    download_id +
    `?om=1` +
    (download_images ? "&download_images=true" : "") +
    (is_paid_story
      ? `&username=${encodeURIComponent(credentials.username)}&password=${encodeURIComponent(credentials.password)}`
      : "") +
    `&mode=${mode}` +
    (download_as_pdf ? "&format=pdf" : "&format=epub");

  $: {
    if (input_url.length) {
      input_url = input_url.toLowerCase();

      invalid_url = false;

      if (/^\d+$/.test(input_url)) {
        // All numbers
        download_id = input_url;
        mode = "story";
      } else if (input_url.includes("wattpad.com/")) {
        // Is a string and contains contain wattpad.com/

        if (input_url.includes("/story/")) {
          // https://wattpad.com/story/237369078-wattpad-books-presents
          input_url = input_url.split("-")[0].split("?")[0].split("/story/")[1]; // removes tracking fields and title
          download_id = input_url;
          mode = "story";
        } else if (input_url.includes("/stories/")) {
          // https://www.wattpad.com/api/v3/stories/237369078?fields=...
          input_url = input_url.split("?")[0].split("/stories/")[1]; // removes params
          download_id = input_url;
          mode = "story";
        } else {
          // https://www.wattpad.com/939051741-wattpad-books-presents-the-qb-bad-boy-and-me
          input_url = input_url
            .split("-")[0]
            .split("?")[0]
            .split("wattpad.com/")[1]; // removes tracking fields and title
          download_id = input_url;
          if (/^\d+$/.test(download_id)) {
            // If "wattpad.com/{download_id}" contains only numbers
            mode = "part";
          } else {
            invalid_url = true;
            input_url = "";
            download_id = "";
          }
        }
      } else {
        invalid_url = true;
      }

      input_url = input_url.match(/\d+/g)?.join("") || "";
      download_id = input_url;

      // Originally, I was going to call the Wattpad API (wattpad.com/api/v3/stories/${story_id}), but Wattpad kept blocking those requests. I suspect it has something to do with the Origin header, I wasn't able to remove it.
      // In the future, if this is considered, it would be cool if we could derive the Story ID from a pasted Part URL. Refer to @AaronBenDaniel's https://github.com/AaronBenDaniel/WattpadDownloader/blob/49b29b245188149f2d24c0b1c59e4c7f90f289a9/src/api/src/create_book.py#L156 (https://www.wattpad.com/api/v3/story_parts/{part_id}?fields=url).
    } else {
      invalid_url = false;
      download_id = "";
    }
  }
</script>

<div>
  <div class="hero min-h-screen">
    <div
      class="hero-content flex-col lg:flex-row-reverse bg-base-100/50 lg:p-16 py-32 rounded shadow-sm"
    >
      {#if !after_download_page}
        <div class="text-center lg:text-left lg:p-10">
          <h1
            class="font-extrabold text-transparent text-5xl bg-clip-text bg-gradient-to-r to-pink-600 via-yellow-600 from-red-700"
          >
            Wattpad Downloader
          </h1>
          <!-- <div role="alert" class="alert bg-cyan-300 mt-5">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              class="h-6 w-6 shrink-0 stroke-current"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
              ></path>
            </svg>
            <span class="text-lg">Please Donate</span>
          </div> -->
          <p class="pt-6 text-lg">
            Download your favourite books with a single click!
          </p>
          <ul class="pt-4 list list-inside text-xl">
            <!-- TODO: 'max-lg: hidden' to hide on screen sizes smaller than lg. I'll do this when I figure out how to make this show up _below_ the card on smaller screen sizes. -->
            <li>12/24 - ⚡ Super-fast Downloads!</li>
            <li>12/24 - 📑 PDF Downloads!</li>
            <li>12/24 - 📂 Improved Performance</li>
            <li>11/24 - 🔗 Paste Links!</li>
            <li>11/24 - 📨 Send to Kindle Support!</li>

            <li>11/24 - ⚒️ Fix Image Downloads</li>
            <li>
              10/24 - 👾 Add the <a
                href="https://discord.com/oauth2/authorize?client_id=1292173380065296395&permissions=274878285888&scope=bot%20applications.commands"
                target="_blank"
                class="link underline">Discord Bot</a
              >!
            </li>
            <li>07/24 - 🔡 RTL Language support! (Arabic, etc.)</li>
            <li>06/24 - 🔑 Authenticated Downloads!</li>
            <li>06/24 - 🖼️ Image Downloading!</li>
          </ul>
        </div>
        <div class="card shrink-0 w-full max-w-sm shadow-2xl bg-base-100">
          <form class="card-body">
            <div class="form-control">
              <input
                type="text"
                placeholder="Story URL"
                class="input input-bordered"
                class:input-warning={invalid_url}
                bind:value={input_url}
                required
                name="input_url"
              />
              <label class="label" for="input_url">
                {#if invalid_url}
                  <p class=" text-red-500">
                    Refer to (<button
                      class="link font-semibold"
                      onclick="StoryURLTutorialModal.showModal()"
                      data-umami-event="Part StoryURLTutorialModal Open"
                      >How to get a Story URL</button
                    >).
                  </p>
                {:else}
                  <button
                    class="label-text link font-semibold"
                    onclick="StoryURLTutorialModal.showModal()"
                    data-umami-event="StoryURLTutorialModal Open"
                    >How to get a Story URL</button
                  >
                {/if}
              </label>

              <label class="cursor-pointer label">
                <span class="label-text"
                  >This is a Paid Story, and I've purchased it</span
                >
                <input
                  type="checkbox"
                  class="checkbox checkbox-warning shadow-md"
                  bind:checked={is_paid_story}
                />
              </label>
              {#if is_paid_story}
                <label class="input input-bordered flex items-center gap-2">
                  Username
                  <input
                    type="text"
                    class="grow"
                    name="username"
                    placeholder="foxtail.chicken"
                    bind:value={credentials.username}
                    required
                  />
                </label>
                <label class="input input-bordered flex items-center gap-2">
                  Password
                  <input
                    type="password"
                    class="grow"
                    placeholder="supersecretpassword"
                    name="password"
                    bind:value={credentials.password}
                    required
                  />
                </label>
              {/if}
            </div>

            <div class="form-control mt-6">
              <a
                class="btn rounded-l-none"
                class:btn-primary={!download_as_pdf}
                class:btn-secondary={download_as_pdf}
                class:btn-disabled={button_disabled}
                data-umami-event="Download"
                href={url}
                on:click={() => (after_download_page = true)}>Download</a
              >

              <label class="swap w-fit label mt-2">
                <input type="checkbox" bind:checked={download_as_pdf} />
                <div class="swap-on">
                  Downloading as <span class=" underline text-bold">PDF</span> (Click)
                </div>
                <div class="swap-off">
                  Downloading as <span class=" underline text-bold">EPUB</span> (Click)
                </div>
              </label>

              <label class="cursor-pointer label">
                <span class="label-text"
                  >Include Images (<strong>Slower Download</strong>)</span
                >
                <input
                  type="checkbox"
                  class="checkbox checkbox-warning shadow-md"
                  bind:checked={download_images}
                />
              </label>
            </div>
          </form>

          <button
            data-feedback-fish
            class="link pb-4"
            data-umami-event="Feedback">Feedback</button
          >
        </div>
      {:else}
        <div class="text-center max-w-4xl">
          <h1 class="font-bold text-3xl">
            Your download has <span
              class="text-transparent bg-clip-text bg-gradient-to-r to-pink-600 via-yellow-600 from-red-700"
              >Started</span
            >
          </h1>
          <div class="py-4 space-y-2">
            <p class="text-2xl">
              If you found this site useful, please consider <a
                href="https://github.com/TheOnlyWayUp/WattpadDownloader"
                target="_blank"
                class="link"
                data-umami-event="Star">starring the project</a
              > to support WattpadDownloader.
            </p>
            <p class="text-lg pt-2">
              You can also join us on <a
                href="https://discord.gg/P9RHC4KCwd"
                target="_blank"
                class="link"
                data-umami-event="Discord">discord</a
              >, where we release features early and discuss updates.
            </p>
          </div>
          <div class="grid justify-center grid-rows-2 gap-y-10">
            <a
              href="/donate"
              target="_blank"
              class="btn bg-cyan-200 btn-lg mt-10 hover:bg-green-200"
              >Buy me a Coffee! 🍵</a
            >
            <button
              on:click={() => {
                after_download_page = false;
                input_url = "";
              }}
              class="btn btn-outline btn-lg">Download More</button
            >
          </div>
        </div>
      {/if}
    </div>
  </div>
</div>

<!-- Open the modal using ID.showModal() method -->

<dialog id="StoryURLTutorialModal" class="modal">
  <div class="modal-box">
    <form method="dialog">
      <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2"
        >✕</button
      >
    </form>
    <h3 class="font-bold text-lg">Finding the Story URL</h3>
    <ol class="list list-disc list-inside py-4 space-y-4">
      <li>
        Copy the URL from the Website, or hit share and copy the URL on the App.
      </li>
      <li>
        For example,
        <span class="font-mono bg-slate-100 p-1"
          >wattpad.com/<span class="bg-amber-200 rounded-sm">story</span
          >/237369078-wattpad-books-presents</span
        >.
      </li>
      <li>
        <span class="font-mono bg-slate-100 p-1"
          >https://www.wattpad.com/939103774-given</span
        > is okay too.
      </li>
      <li>Paste the URL and hit Download!</li>
    </ol>
  </div>
  <form method="dialog" class="modal-backdrop">
    <button>close</button>
  </form>
</dialog>
