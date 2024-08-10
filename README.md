# ticket.js
```
const Discord = require('discord.js');
const { permissionsEmbed, TickEmbed } = require('../../client/embed.js');
const ticket = require('../../database/models/ticket');

module.exports = {
    name: 'ticket',
    aliases: ['bilet'],

    async execute(client, message, args) {
        if (!message.member.permissions.has(Discord.PermissionFlagsBits.Administrator)) {
            return message.channel.send(permissionsEmbed(`# <:cancel:1266880050494705685> Bir hata aldım!\nMerhaba **${message.author.username}**, malesefki <:alert:1266881899192647803> **yönetici** yetkiniz bulunmadığından komutuna erişemiyorsunuz.`));
        }

        const roleOptions = message.guild.roles.cache.map(role => ({
            label: role.name,
            value: role.id,
            emoji: `<:roles2:1267139852005019668>`
        })).slice(0, 25);
        
        const roleSelectMenu = new Discord.StringSelectMenuBuilder()
            .setCustomId('roleSelectMenu')
            .setPlaceholder('Lütfen bir rol seçin')
            .addOptions(roleOptions);
        
        const selectMenuRow = new Discord.ActionRowBuilder().addComponents(roleSelectMenu);
        
        const embed = new Discord.EmbedBuilder()
            .setColor(process.env.MAIN_COLOR || '#5865F2')
            .setDescription('<:rolemodel:1267136680897282068> Bir **Yetkili Rol** girmen gerekiyor. Bunun için aşağıdaki menüyü kullan!');
        
        const replyMessage = await message.reply({
            embeds: [embed],
            components: [selectMenuRow]
        });
        
        const roleFilter = (interaction) => interaction.customId === 'roleSelectMenu' && interaction.user.id === message.author.id;
        const roleCollector = replyMessage.createMessageComponentCollector({ filter: roleFilter, time: 60000 });
        
        roleCollector.on('collect', async (interaction) => {
            try {
                const selectedRoleId = interaction.values[0];
                const selectedRole = message.guild.roles.cache.get(selectedRoleId);
        
                if (!selectedRole) {
                    return interaction.reply({ content: 'Seçilen rol bulunamadı.', ephemeral: true });
                }

                const channelOptions = message.guild.channels.cache.filter(channel => channel.type === Discord.ChannelType.GuildText).map(channel => ({
                    label: channel.name,
                    value: channel.id,
                    emoji: `<:text_channel:1267138418349965355>`
                })).slice(0, 25);
        
                const channelSelectMenu = new Discord.StringSelectMenuBuilder()
                    .setCustomId('channelSelectMenu')
                    .setPlaceholder('Kanal Seç')
                    .addOptions(channelOptions);
        
                const channelSelectMenuRow = new Discord.ActionRowBuilder().addComponents(channelSelectMenu);
        
                const updatedEmbed = new Discord.EmbedBuilder()
                    .setColor(process.env.MAIN_COLOR || '#5865F2')
                    .setDescription('<:text:1266882169499029625> Bir **Kanal** girmen gerekiyor. Bunun için aşağıdaki menüyü kullan!');
        
                await interaction.update({
                    embeds: [updatedEmbed],
                    components: [channelSelectMenuRow]
                });
        
                const channelFilter = (i) => i.customId === 'channelSelectMenu' && i.user.id === message.author.id;
                const channelCollector = replyMessage.createMessageComponentCollector({ filter: channelFilter, time: 60000 });
        
                channelCollector.on('collect', async (channelInteraction) => {
                    try {
                        const selectedChannelId = channelInteraction.values[0];
                        const selectedChannel = message.guild.channels.cache.get(selectedChannelId);
        
                        if (!selectedChannel) {
                            return channelInteraction.reply({ content: 'Seçilen kanal bulunamadı.', ephemeral: true });
                        }

                        const categoryOptions = message.guild.channels.cache.filter(channel => channel.type === Discord.ChannelType.GuildCategory).map(category => ({
                            label: category.name,
                            value: category.id,
                            emoji: `<:role_req:1267139964655763477>`
                        })).slice(0, 25);
        
                        const categorySelectMenu = new Discord.StringSelectMenuBuilder()
                            .setCustomId('categorySelectMenu')
                            .setPlaceholder('Kategori Seç')
                            .addOptions(categoryOptions);
        
                        const categorySelectMenuRow = new Discord.ActionRowBuilder().addComponents(categorySelectMenu);
        
                        const updatedEmbedWithChannel = new Discord.EmbedBuilder()
                            .setColor(process.env.MAIN_COLOR || '#5865F2')
                            .setDescription('Bir **Kategori** girmen gerekiyor. Bunun için aşağıdaki menüyü kullan!');
        
                        await channelInteraction.update({
                            embeds: [updatedEmbedWithChannel],
                            components: [categorySelectMenuRow]
                        });
        
                        const categoryFilter = (i) => i.customId === 'categorySelectMenu' && i.user.id === message.author.id;
                        const categoryCollector = replyMessage.createMessageComponentCollector({ filter: categoryFilter, time: 60000 });
        
                        categoryCollector.on('collect', async (categoryInteraction) => {
                            try {
                                const selectedCategoryId = categoryInteraction.values[0];
                                const selectedCategory = message.guild.channels.cache.get(selectedCategoryId);
        
                                if (!selectedCategory) {
                                    return categoryInteraction.reply({ content: 'Seçilen kategori bulunamadı.', ephemeral: true });
                                }
        
                                const finalEmbed = new Discord.EmbedBuilder()
                                    .setColor(process.env.SUCCESS_COLOR || '#57F287')
                                    .setThumbnail('https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExMjg4Ynh3dzY4OGl2bHFpbDlmazdsbDRrZjl4dDlpbjc2ZGw3N3N5ZyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/MCQOHYCZAwGAh25OeP/giphy.gif')
                                    .setDescription(`## Başarıyla Tamamlandı!
        **Ayarlananlar;**
        ● Ticket Kanal: **${selectedChannel.name}**
        ● Yetkili Rol: **${selectedRole.name}**
        ● Ticket Kategori: **${selectedCategory.name}**`)
                                    .setFooter({ text: `"@${process.env.BOT_NAME}" rolünü en üste almayı unutma.`});
        
                                await categoryInteraction.update({
                                    embeds: [finalEmbed],
                                    components: []
                                }).then(async (a) => {setTimeout(async () => {a.delete()}, 10000)})
        
                                const aktifValue = `<:aktif1:1267149207513468999>`;
                                const existingDocument = await ticket.findOne({ guild: message.guild.id });
        
                                if (!existingDocument) {
                                    const newDocument = new ticket({
                                        guild: message.guild.id,
                                        role: selectedRole.id,
                                        channel: selectedChannel.id,
                                        category: selectedCategory.id
                                    });
        
                                    await newDocument.save();
                                } else {
                                    await ticket.findOneAndUpdate({ guild: message.guild.id }, {
                                        role: selectedRole.id,
                                        channel: selectedChannel.id,
                                        category: selectedCategory.id
                                    });
                                }
        
                                selectedChannel.send({
                                    embeds: [
                                        new Discord.EmbedBuilder()
                                            .setColor(process.env.MAIN_COLOR || '#5865F2')
                                            .setDescription(`# Ticket Menüsü\nTicket açmak için "<:role_change:1267139830710534155>" buttonuna basmanız yeterli olucaktır.`)
                                            .setFooter({ text: `${process.env.BOT_NAME}`, iconURL: client.user.avatarURL({ size: 2048 }) })
                                            .setTimestamp()
                                    ],
                                    components: [
                                        new Discord.ActionRowBuilder().addComponents(
                                            new Discord.ButtonBuilder()
                                                .setCustomId('ticketac')
                                                .setEmoji('<:role_change:1267139830710534155>')
                                                .setStyle(Discord.ButtonStyle.Secondary)
                                        )
                                    ]
                                });
                            } catch (error) {
                                console.error('Kategori seçimi sırasında bir hata oluştu:', error);
                            }
                        });
                    } catch (error) {
                        console.error('Kanal seçimi sırasında bir hata oluştu:', error);
                    }
                });
            } catch (error) {
                console.error('Rol seçimi sırasında bir hata oluştu:', error);
            }
        });
        
        roleCollector.on('end', collected => {
            if (collected.size === 0) {
                /*replyMessage.edit({
                    components: [selectMenuRow.setDisabled(true)]
                });*/
            }
        });
    },
};
```

# ticket-button.js

```
const Discord = require('discord.js');
const ticket = require('../../database/models/ticket.js');

module.exports = (client) => {
    client.on('interactionCreate', async (interaction) => {
        if (interaction.isButton()) {
            if (interaction.customId === 'ticketac') {

                const existingTicket = await ticket.findOne({
                    guild: interaction.guild.id,
                    userId: interaction.user.id,
                    aktif: true // Sadece aktif olan ticket'ları kontrol et
                });

                if (existingTicket) {
                    const ticket_acamama = new Discord.EmbedBuilder()
                        .setColor(process.env.MAIN_COLOR)
                        .setDescription(`<:close:1269596239498842163> Zaten aktif bir ticket'ınız var. Yeni bir ticket oluşturamazsınız.`)
                    return interaction.reply({
                        embeds: [ticket_acamama],
                        ephemeral: true
                    });
                }

                const modal = new Discord.ModalBuilder()
                    .setCustomId('ticketModal')
                    .setTitle('Ticket Oluşturma Sebebi');

                const reasonInput = new Discord.TextInputBuilder()
                    .setCustomId('reasonInput')
                    .setLabel('Sebep')
                    .setStyle(Discord.TextInputStyle.Paragraph)
                    .setRequired(true)
                    .setMaxLength(500);

                const actionRow = new Discord.ActionRowBuilder().addComponents(reasonInput);
                modal.addComponents(actionRow);

                await interaction.showModal(modal);
            }

            if (interaction.customId.startsWith('delete-')) {
                const channelId = interaction.customId.split('-')[1];
                const channel = client.channels.cache.get(channelId);

                if (channel) {
                    const ticketEntry = await ticket.findOne({ channelId: channel.id });

                    if (ticketEntry) {
                        const openTime = ticketEntry.createdAt;
                        const closeTime = new Date();
                        const duration = closeTime - openTime;
                        const durationSeconds = Math.floor(duration / 1000);
                        const durationMinutes = Math.floor(duration / (1000 * 60));
                        const durationHours = Math.floor(duration / (1000 * 60 * 60));

                        const messages = await channel.messages.fetch();
                        const messageCount = messages.size;

                        const reason = ticketEntry.reason || 'Sebep belirtilmemiş';

                        await channel.delete().catch(console.error);

                        const ticketData = await ticket.findOne({ guild: interaction.guild.id });
                        if (ticketData) {
                            const role = interaction.guild.roles.cache.get(ticketData.role);
                            if (role) {
                                role.members.forEach(member => {
                                    member.send({
                                        content: `Merhaba ${member}, ${interaction.user.tag} adlı kullanıcı ticket'ını sildi.`
                                    }).catch(console.error);
                                });
                            }
                        }

                        try {
                            await interaction.user.send({
                                embeds: [
                                    new Discord.EmbedBuilder()
                                        .setColor(process.env.MAIN_COLOR)
                                        .setDescription(`<:roleplaying1:1267136742604144711> Merhaba **${interaction.user.tag}**, ticket silindi.\n\n<:delete2:1267983669767831643> **Kapatan kullanıcı:** ${interaction.user.tag} [${interaction.user.id}]\n<:24h:1266881919962845295> **Açılış süresi:** ${durationHours} saat ${durationMinutes} dakika ${durationSeconds} saniye\n<:message:1266882152335671498> **Kanal mesaj sayısı:** ${messageCount}\n<:delete:1267983718233014414> **Siliniş tarihi:** <t:${Math.floor(Date.now() / 1000)}:R>`)
                                        .setFooter({ text: `${interaction.client.user.username}`, iconURL: interaction.client.user.avatarURL({ size: 2048 }) })
                                ]
                            }).catch(console.error);
                        } catch (sendError) {
                            console.error('Kullanıcıya mesaj gönderme hatası:', sendError);
                        }

                        await ticket.deleteOne({ userId: ticketEntry.userId, guild: ticketEntry.guild, channelId: channel.id }).catch(console.error);
                    } else {
                        console.error('Ticket verisi bulunamadı.');
                    }
                } else {
                    console.error('Kanal bulunamadı.');
                }
            }
        }

        if (interaction.isModalSubmit()) {
            if (interaction.customId === 'ticketModal') {
                const reason = interaction.fields.getTextInputValue('reasonInput');
                const userId = interaction.user.id;

                const guild = interaction.guild;
                const ticketData = await ticket.findOne({ guild: guild.id });

                if (!ticketData) {
                    return interaction.reply({ content: 'Ticket sistemi ayarlanmamış.', ephemeral: true });
                }

                const category = guild.channels.cache.get(ticketData.category);
                if (!category) {
                    return interaction.reply({ content: 'Kategori bulunamadı.', ephemeral: true });
                }

                const channel = await guild.channels.create({
                    name: `ticket-${userId}`,
                    type: Discord.ChannelType.GuildText,
                    parent: category.id,
                    permissionOverwrites: [
                        {
                            id: guild.id,
                            deny: [Discord.PermissionFlagsBits.ViewChannel],
                        },
                        {
                            id: userId,
                            allow: [
                                Discord.PermissionFlagsBits.ViewChannel,
                                Discord.PermissionFlagsBits.SendMessages,
                                Discord.PermissionFlagsBits.ReadMessageHistory,
                            ],
                        },
                        {
                            id: ticketData.role,
                            allow: [
                                Discord.PermissionFlagsBits.ViewChannel,
                                Discord.PermissionFlagsBits.SendMessages,
                                Discord.PermissionFlagsBits.ReadMessageHistory,
                            ],
                        },
                    ],
                });

                await ticket.create({
                    channelId: channel.id,
                    createdAt: new Date(),
                    guild: guild.id,
                    category: category.id,
                    role: ticketData.role,
                    reason: reason,
                    userId: userId,
                    aktif: true // Yeni ticket aktif olarak işaretlenir
                }).catch(console.error);

                const embed = new Discord.EmbedBuilder()
                    .setColor(process.env.MAIN_COLOR)
                    .setDescription(`# <:roleplaying:1267136755765612677> Bir ticket oluşturuldu!`)
                    .addFields(
                        { name: `<:rolemodel:1267136680897282068> Ticketi Oluşturan;`, value: `${interaction.user.tag} [${interaction.user.id}]`, inline: true },
                        { name: `Ticketin Oluşturulma Tarihi;`, value: `<t:${Math.floor(Date.now() / 1000)}:R>` },
                        { name: `Ticketin Oluşturulma Sebebi;`, value: `\`\`\`${reason}\`\`\``, inline: true },
                    );

                const message = await channel.send({
                    embeds: [embed],
                    components: [
                        new Discord.ActionRowBuilder()
                            .addComponents(
                                new Discord.ButtonBuilder()
                                    .setCustomId(`delete-${channel.id}`)
                                    .setLabel('Kanalı Sil')
                                    .setEmoji('<:trash:1267983692131995669>')
                                    .setStyle(Discord.ButtonStyle.Danger)
                            )
                    ]
                });
                await message.pin().catch(console.error);

                const ticket_okey_embed = new Discord.EmbedBuilder()
                    .setColor(process.env.MAIN_COLOR)
                    .setDescription(`<:nashira_kapali:1267149239759409224> Ticket oluşturuldu: <#${channel.id}>`);
                await interaction.reply({ embeds: [ticket_okey_embed], ephemeral: true });

                await interaction.user.send({
                    embeds: [
                        new Discord.EmbedBuilder()
                            .setColor(process.env.MAIN_COLOR)
                            .setDescription(`<:roleplaying1:1267136742604144711> Merhaba **${interaction.user.tag}**, ticket'iniz oluşturuldu.\n\n<:calendarpage:1266881869954420860> **Açılış tarihi:** <t:${Math.floor(Date.now() / 1000)}:R>\n<:Roles3:1267139820266979338> **Sebep:**\n\`\`\`${reason}\`\`\``)
                            .setFooter({ text: `${interaction.client.user.username}`, iconURL: interaction.client.user.avatarURL({ size: 2048 }) })
                    ]
                }).catch(console.error);
            }
        }
    });
};
```


