E/RecyclerView: No adapter attached; skipping layout

Solution - 
with(dialog.recyclerView) {
                layoutManager = LinearLayoutManager(this.context)
                setHasFixedSize(true)
                adapter = DeliverySettingsAdapter(items)
            }