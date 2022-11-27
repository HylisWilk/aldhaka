# The Botanical Codex of Aldhaka
Nanogenmo entry for 2022

# Description
## Algorithm Breakdown
### Flower names

Flower names are generated as binomial species names. They consist on a genus and and adjective.
I've hand picked 25 options for each, which generates 625 possible flower species. 
The genus choices are mostly just flowers that I like or thought might look pretty as botanical illustrations (lotuses, bromeliads, orchids, birds of paradise, etc.)
The adjectives are just random ones that I saw in other flower species or random words I translated to latin on google translate, wondering what would happen.

The possible genus words are: Nelumba, Magnolia, Strelitzia, Plumeria, Passiflora, Heliconia, Helianthus, Alstroemeria, Convallaria, Crocus, Cattleya, Calendula, Dahlia, DelphiniumIris, Jasminum, Malva, Gardenia, Petunia, Lilium, Lobelia, Lamprocapnos, Hydrangea, Guzmania, Leontopodium, Luzuriaga.

The possible adjective words are: Rubra, Caerulea, Viridis, Viridiflora, Alba, Rosea, Aurea, Japonica, Amazonensis, Italicum, Americana, Sinensis, Germanica, Amakusaensis, Anatoliensis, Andalusiensis, Asturiensis, Bengalensis, Canariensis, Pulcherrima, Atrox, Diamondi, Purpurata, Terrificus, Darwinii

I'm nearly sure that none of the resulting generated species actually exist.

### Flower descriptions

Once we have a species name (for instance 'Nelumba Rubra'), we generate a description for this species. This step is done using the OPT 1.3b model from Hugging Face. I would've prefered using GPT-3 for it but wanted to keep it as open source as possible. The OPT outputs were reasonably alright with not too much prompt engineering.
Speaking of which, after manually trying a few prompts, this one gave me pretty consistent results:

```
prompt = f'Summarized description of the characteristics of the newly found flower species {name}. (Field guide of Botany, Hylis et al. 2022):'
```

Using a relatively long prompt like this where only the name of the flower changes, one can also set a seed every time the code is run and you'll obtain sentences that start very similarly and only change the details of the flowers. This has the side effect of making the final text look the same in the beginning words, but that didn't bother me too much for now. One way to go around this is simply not using seeds and getting more variation. Another would be to generate longer outputs (changing the generation parameters on OPT, but longer texts started including very random things like doi links and often verged off into weird territory, so I opted out of it).

The output of the model will look something like:

```
</s>Summarized description of the characteristics of the newly found flower species Nelumba Rubra. (Field guide of Botany, Hylis et al. 2022):
3 Flower shape is pepulated white sepals surrounding to black lobes around petals which extend from stem edge. A small area appears as yellow patch toward crown margin - usually between four with very wide angle branches spreading 1‚15 cm over 6¼cm plant body that produces about 12-29 bloom
```

### Flower images

Now we need to pass this description somehow to Stable Diffusion 1.5, to generate the images.
I used the great SD 1.5 webui colab by Camenduru, many thanks to him. https://github.com/camenduru/stable-diffusion-webui

Within the SD webui there's an option for generating a batch of prompts. I used 64 sampling steps with the Euler sampler.
The prompt for generating the images was:
```
Botanical Illustration of {flower_name} by Pierre François Turpin. Context:{generated_description}
```
Where flower_name and generated_description were the outputs generated in the previous steps. Pierrre François Turpin is the name of a botanist illustrator I found on Google.

After this step we have all the flower images ready.

### Generating a computer font

This step was the messiest and most hand-crafted. For a future iteration it'd be great to automatize it more. I basically used Midjourney to generate the following prompt:

```
Letter 'A' in a newly found never seen before script --testp --upbeta --creative
```

And rerolled a few times until I got something I liked. Then I generated enough variations of that image to get a symbol for each lower case alphanumeric character (a through z, 0 through 9).

As an example, this is what the output of Midjourney looked like:

![image](https://user-images.githubusercontent.com/88170094/204150569-77bc2238-be13-41ad-bd24-eb31fcebed7b.png)

The images were all added together and cleaned. Ex.:

![image](https://user-images.githubusercontent.com/88170094/204150745-3f9d727c-4d6b-4aec-ae82-3a0e03de11d6.png)

And then they were vectorized and passed onto FontForge to generate aldhaka.ttf, but that was quite a pain to do, very manual, and the process of vectorization lost a lot of the finer details of the curves. I wonder if there is a more automatized way to do it.
Because I only generated images for english alphanumeric characters, I filter out everything else from the final generated pages. This mangles the descriptions a bit but not too much in general.

In the end though, we are capable of generating full pages of text using the generated_text from previous steps, as such:

![page0](https://user-images.githubusercontent.com/88170094/204150846-9b92673e-d0b9-40fa-9fe5-c710a5ded77d.png)

### Stylizing the page

The thing with those pages is that every single letter is completely uniform and looks exactly the same every time it is written, which is not very human like. To impart some variation I threw all generated pages onto Stable Diffusion img2img with the prompt 'Handwritten ancient papyrus' with very low CFG and for only a few sampling steps, and the result is:

![index](https://user-images.githubusercontent.com/88170094/204150999-053cc35d-5807-4941-be96-3c3a760e72e0.png)

There's slight variation now every time a given letter is used, although it loses a bit of image quality. Still, seems like a decent proof of concept.

### Putting it toghether

We basically just horizontally stack the two image arrays:

![index](https://user-images.githubusercontent.com/88170094/204151066-e0feae64-6965-4373-89b7-ed9c553dc2b8.png)

And save it as a pdf with PIL.

### Cover

The cover was generated on Midjourney with the prompt

```
Manga cover, great medieval codex grimoire of magic flowers, persian art --ar 2:3 --test --creative
```

And a variation to make it 2 images.

## Todo

- I'd still like to improve the page layout to something more interesting than just hstacking, but I might leave that for later.
- Trying to add a papyrus or old book texture to the text pages with SD or VQGAN+CLIP destroys the text. Maybe I can do that in the future with simpler generative art techniques.
