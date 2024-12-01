<script>
  import Quagga from "quagga";
  import {
    Button,
    Input,
    Table,
    Jumbotron,
    InputGroup,
    InputGroupAddon,
    Card,
  } from "sveltestrap";

  import {
    FileEarmarkArrowUpFill,
    UpcScan,
    EnvelopeFill,
  } from "svelte-bootstrap-icons";

  // let catalog

  // fetch("https://cors-anywhere.herokuapp.com/https://westmoorsstores.s3.eu-west-2.amazonaws.com/catalog.json")
  // .then(res => res.json())
  // .then((data) => catalog = data)

  import catalog from "./catalog.json";

  // Should work with Hot Chocolate

  let scannerRef;
  let isRunning = false;

  let d = new Date();
  let when = d.toUTCString();

  $: list = [
    {
      img_small_url:
        "https://www.booker.co.uk/bbimages/1/071ad5a2-e072-400a-bae6-93e99ae88451.jpg",
      name: "(EXAMPLE) Corona Lager",
      code: 135253,
      barcode: 75041670,
      quantity: 2,
    },
  ];

  $: file = list
    .map((p) => ["10/01/20,00:00:00", p.quantity, p.barcode + "%0D%0A"])
    .join("");

  function startScanner() {
    Quagga.init(
      {
        inputStream: {
          name: "Live",
          type: "LiveStream",
          target: scannerRef,
          constraints: {
            width: 480,
            height: 320,
            facingMode: "environment",
          },
        },
        decoder: {
          readers: ["upc_reader", "upc_e_reader"],
        },
      },
      (err) => {
        if (err) {
          alert(err);
          return;
        }

        Quagga.video.dom.overlay.style.display = "none";
        Quagga.start();
        isRunning = true;
      }
    );

    Quagga.onDetected((result) => add(result.codeResult.code));
  }

  function add(barcode) {
    let product = catalog.find((p) => p.barcode === barcode);
    list = [...list, { ...product, ...{ quantity: 1 } }];
  }
</script>

<main>
  <div class="container" style="padding: 0 1rem; max-width: 50rem">
    <Jumbotron class="mt-5">
      <h1>Booker Scanner</h1>
      <p class="lead">The free mobile scanner for Booker customers</p>
      <hr class="my-2" />
      <p>
        Think you need to buy an expensive scanner for purchase orders and stock
        takes? Think again. Scan products using your phone, enter a quantity and
        email yourself a file that can be uploaded to the Booker website to
        create a shopping list or imported into Excel to do stock takes.
      </p>
    </Jumbotron>
    <Card body>
      <div bind:this={scannerRef} />
      <Button
        color="primary"
        class="mr-2"
        on:click={() => (isRunning ? Quagga.stop() : startScanner())}
      >
        <UpcScan />
        Toggle Scanner
      </Button>
    </Card>
    <Table responsive borderless class="mt-3">
      <thead>
        <tr>
          <th />
          <th>Quantity</th>
          <th />
          <th>Code</th>
        </tr>
      </thead>
      <tbody>
        {#each list as { img_small_url, name, code, quantity }}
          <tr>
            <td>
              <img src={img_small_url} style="width: 2rem" alt />
            </td>
            <td>
              <InputGroup>
                <Input
                  size="sm"
                  style="width: 1.5rem"
                  number
                  bind:value={quantity}
                />
                <InputGroupAddon addonType="append">
                  <Button
                    on:click={() => quantity++}
                    outline
                    size="sm"
                    color="secondary"
                  >
                    <strong>+</strong>
                  </Button>
                  <Button
                    on:click={() => {
                      quantity === 1
                        ? (list = list.filter((p) => p.code !== code))
                        : quantity--;
                    }}
                    outline
                    size="sm"
                    color="secondary"
                  >
                    <strong>-</strong>
                  </Button>
                </InputGroupAddon>
              </InputGroup>
            </td>
            <td>
              <a
                href="https://www.booker.co.uk/catalog/productinformation.aspx?code={code}"
                target="_blank"
              >
                {name}</a
              >
            </td>

            <td>{code}</td>
          </tr>
        {/each}
      </tbody>
    </Table>
    <div class="btn-toolbar float-right mb-3" role="toolbar">
      <form
        action="mailto:your email?subject=Sammy Scanner File ({when})&body={file}"
        method="post"
        class="mr-2"
      >
        <Button color="success" type="submit">
          <EnvelopeFill />
          Email File
        </Button>
      </form>

      <form
        action="https://www.booker.co.uk/catalog/scanneruploadcs3000.aspx"
        target="_blank"
        class="mr-2"
      >
        <Button color="secondary" type="submit">
          <FileEarmarkArrowUpFill />
          Upload
        </Button>
      </form>
    </div>
  </div>
</main>
